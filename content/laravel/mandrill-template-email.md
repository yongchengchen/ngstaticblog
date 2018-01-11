<!--
Categories = ["Development", "Laravel"]
Description = "Laravel Sending Email using Mandrill Template"
Tags = ["Development", "laravel", "Mandrill", "Template"]
date = "2018-01-11T21:47:31-08:00"
title = "Laravel Sending Email using Mandrill Template"
-->

## Laravel Sending Email using Mandrill Template

Laravel has supported many Email service transport which also support Mandrill. But it doesn't allow we use Mandrill email template directly.

Here's a simple solution to support Mandrill email template by only add two classes.

#### 1. Class MandrillTransport

```php
<?php

namespace Yong\MandrillMail\Model;

use Mandrill;

use Illuminate\Mail\Mailable;
use Carbon\Carbon;

class MandrillTransport
{
    private static $self;

    public static function singleton(...$args) {
        if (empty(self::$self)) {
            self::$self = new static(...$args);
        }
        return self::$self;
    }

    protected $api;
    protected function __construct() {
        $this->api = new Mandrill(config('services.mandrill.secret'));
    }

    public function sendTemplate($template_name, $template_content, $message, $send_at=null) {
        return $this->api->messages->sendTemplate($template_name, $template_content, $message, $this->isAsync(), $this->ip_pool(), $send_at);
    }

    protected function isAsync() {
        return config('services.mandrill.async');
    }

    protected function ip_pool() {
        return config('services.mandrill.ip_pool');
    }
}
```
#### 2. trait TraitMailChimpMessage

```php
<?php

namespace Yong\MandrillMail\Model\Messages;

use Illuminate\Contracts\Mail\Mailer as MailerContract;
use Yong\MandrillMail\Model\MandrillTransport;

// https://mandrillapp.com/api/docs/messages.html

trait TraitMailChimpMessage
{
    protected $mailchimpArguments;
    public function __sleep() {
        $this->mailchimpArguments = $this->build();
        return parent::__sleep();
    }

    protected function getMailChimpArguments() {
        if (empty($this->mailchimpArguments)) {
            $this->mailchimpArguments = $this->build();
        }
        return $this->mailchimpArguments;
    }
    /**
     * Send the message using the given mailer.
     *
     * @param  \Illuminate\Contracts\Mail\Mailer  $mailer
     * @return void
     */
    public function send(MailerContract $mailer) {
        $arguments = $this->getMailChimpArguments();
        MandrillTransport::singleton()->sendTemplate(
            $this->template,
            $arguments['content'],
            $arguments['message'],
            null
        );
    }
    
    protected function getTemplateMessageCommonParts($from, $reply_to=null, $cc=[], $bcc=[]) {
        $messages = [];
        $messages['from_email'] = Store::getConfig(static::FROM);
        $messages['from_name'] = Store::getConfig(static::FROM_NAME, $messages['from_email']);
        $messages['track_opens'] = true;
        $messages['track_clicks'] = true;
        $messages['merge_language'] = 'handlebars';

        if (count($cc) > 0) {
            $messages['to'] = [[
                'email'=>$cc[0],
                'name'=>$cc[1]
                'type'=>'cc'
            ]];
        }
        
        if (!empty($reply_to)) {
            $messages['headers'] = ['Reply-To' => $reply_to];
        }
        if (property_exists($this, 'important') && $this->important) {
            $messages['important'] = $this->important;
        }

        if (count($bcc) > 0) {
            $messages['bcc_address'] = $bcc[0];
        }

        $messages['global_merge_vars'] = $this->getCustomMergeVars();
        return $messages;
    }
}
```

### How to use

#### 1. Create an email message
```php
<?php

namespace Yong\MandrillMail\Model\Messages;

use Illuminate\Bus\Queueable;
use Illuminate\Mail\Mailable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Contracts\Queue\ShouldQueue;
use Yong\MandrillMail\Model\TraitMailChimpMessage;

class TestMessage extends Mailable
{
    use Queueable, SerializesModels;
    use TraitMailChimpMessage;

    protected $template = 'your_mandrill_template_name';

    protected $params;
    function __construct($params)
    {
        $this->params = $params;
    }

    function build() {
        return [
            'content' => $this->getTemplateContent(),
            'message' => $this->getTemplateMessage()
        ];
    }

    protected function getTemplateContent() {
        return [];
    }

    protected function getCustomMergeVars() {
        $collection = [];

        $collection[] = [
            'name' => 'Your_mandrill_template_param1',
            'content' => 'value1'
        ];
        $collection[] = [
            'name' => 'Your_mandrill_template_param2',
            'content' => 'value2'
        ];
       
        /* add more parameters here */
        return $collection;
    }

    protected function getTemplateMessage() {
        $messages = $this->getTemplateMessageCommonParts();

        $messages['subject'] = 'subject here';
        $messages['to'][] = [
            'email'=>'to_email_address',
            'name'=>'to_email_name',
            'type'=>'to'
        ];

        return $messages;
    }
}
```

#### 1. send the email message

```php
    use Mail;
    //...
    
    //send email
    Mail::to('test@test.test')->send(new TestMessage($params));

    //or queue email
    Mail::queue(new TestMessage($params));
```
