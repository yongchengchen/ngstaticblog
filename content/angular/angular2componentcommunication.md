<!-- 
Categories = ["Development", "Angular"]
Description = ""
Tags = ["Development", "Angular 2/4"]
date = "2017-09-26T21:47:31-08:00"
title = "Angular 2/4 Communicate Between Components"
-->

## Angular 4 Communicating Between Components

#### Why doing this?

This post shows how to communicate between components in Angular 2/4.

The solution is to use an Observable and a Subject (which is a type of observable).

And the service(MQService) supports multiple nodes to commucate with multiple nodes.

And the node can be component or service.

#### Implement

Here's the source code of MQService. It defines

```typescript
import { Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { Subject } from 'rxjs/Subject';

export interface MQ_MSG_BODY {
    type:string,
    from?:string,
    callback?:(ret:any):void,
    data:any,
}

interface MQ_NODE {
    [key:string]:Subject<MQ_MSG_BODY>
}

@Injectable()
export class MQService {
    protected static nodes:MQ_NODE;

    constructor() {
        MQService.nodes = {};
    }

    notify(node: string, msg_body:MQ_MSG_BODY) {
        this.getNode(node).next(msg_body);
    }

    listen(node: string): Observable<MQ_MSG_BODY> {
        return this.getNode(node).asObservable();
    }
 
    clear() {
        for(let node in MQService.nodes) {
            MQService.nodes[node].next();
        }
    }

    deleteNode(node:string):void {
        if (MQService.nodes[node] != undefined) {
            delete MQService.nodes[node];
        }
    }

    private getNode(node:string):Subject<MQ_MSG_BODY> {
        if (MQService.nodes[node] === undefined) {
            MQService.nodes[node] = new Subject<MQ_MSG_BODY>();
        }
        return MQService.nodes[node];
    }
}
```

#### How To Use

##### 1. Provide it from your root module(AppModule) of your Angular project

```typescript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { MQService } from './mq.service';

@NgModule({
    declarations: [
        AppComponent
    ],
    imports: [
        BrowserModule,
        AppRoutingModule
    ],
    providers: [
        MQService
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

##### 2. Use it in the component

Only 2 steps to init it in your component/service.

(1). import MQService

```typescript
  import { MQService } from './mq.service';
```

(2). init it in your component/service's constructor

```typescript
  constructor(private mq:MQService) { }
```

##### 3. Define receiver in the component

If you define a receiver you need to implement OnInit and OnDestroy

And put listening logic to MQ listen with your MQ node name.

Once it receive a message and it will trigger the listening logic to work.

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Observable } from 'rxjs';
import { Subscription } from 'rxjs/Subscription';
import { MQService } from './mq.service';

@Component({
  selector: 'app-mq-receiver',
  templateUrl: './mq.receiver.component.html',
  styleUrls: ['./mq.receiver.component.scss']
})
export class MQReceiverComponent implements OnInit, OnDestroy {
  private mqSubs:Subscription;

  constructor(private mq:MQService) { }
  
  ngOnInit() {
    this.mqSubs = this.mq.listen('app-mq-receiver').subscribe(
      msg_body => {
        if(msq_body.type == 'something') {
          //...do some thing
        }
        console.log(msg_body);
      }
    );
  }

  ngOnDestroy() {
      this.mqSubs.unsubscribe();
  }
}
```

##### 4. Sending message to  receiver component

Sending message is very easy, you just need to know your target's MQ node name and you can send the message.

```typescript
import { Component } from '@angular/core';
import { Observable } from 'rxjs';
import { MQService } from './mq.service';

@Component({
  selector: 'app-mq-sender',
  templateUrl: './mq.sender.component.html',
  styleUrls: ['./mq.sender.component.scss']
})
export class MQSenderComponent {

  constructor(private mq:MQService) { }
  
  sendMessage() {
    this.mq.notify('app-mq-receiver', {type:'test', data:{ msg:"this is a test" }});
  }
}
```

##### * A component can be both receiver and sender

You can define component as a full MQ node with both receiver and sender

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { Observable } from 'rxjs';
import { Subscription } from 'rxjs/Subscription';
import { MQService } from './mq.service';

@Component({
  selector: 'app-mq-receiver',
  templateUrl: './mq.receiver.component.html',
  styleUrls: ['./mq.receiver.component.scss']
})
export class MQReceiverComponent implements OnInit, OnDestroy {
  private mqSubs:Subscription;

  constructor(private mq:MQService) { }
  
  ngOnInit() {
    this.mqSubs = this.mq.listen('app-mq-receiver').subscribe(
      msg_body => {
        if(msq_body.type == 'something') {
          //...do some thing
        }
        console.log(msg_body);
      }
    );
  }

  ngOnDestroy() {
      this.mqSubs.unsubscribe();
  }

  sendMessage() {
    this.mq.notify('other_node', {type:'test', data:{ msg:"this is a test" }});
  }
}
```

#### Examples

By using MQService, you can easily define a common module/modal/component by receiving message and response to many request.

Loader layer is a simple use of MQService.

##### LoaderLayerComponent

* loaderlayer.component.html
```html
<div class="loading-mask" *ngIf="loading">
    <div class="loader">
        <img alt="Loading..." [src]="/assets/images/loader.gif" />
    </div>
</div>
```

* loaderlayer.component.ts
```typescript
import { Component, Input, OnInit, OnDestroy } from '@angular/core';
import { Subscription } from 'rxjs/Subscription';
import { MQService } from '@ngxsuit';

@Component({
    selector: 'app-loaderlayer',
    templateUrl: './loaderlayer.component.html',
    styleUrls: ['./loaderlayer.component.scss']
})

export class LoaderLayerComponent implements OnInit, OnDestroy{
    @Input() loading: boolean;

    private mqSubs: Subscription;

    constructor(private mq:MQService) {}
    
    ngOnInit() {
        this.mqSubs = this.mq.listen('app-loaderlayer').subscribe(msg_body => { 
          swith(msg_body.type) {
            case 'show':
              this.loading = true;
              break;
            case 'hide':
              this.loading = false;
          }
        });
    }

    ngOnDestroy() {
        this.mqSubs.unsubscribe();
    }
}
```

* loaderlayer.component.scss
```css
:host {
    .loading-mask {
        background: rgba(255, 255, 255, 0.5);
    }
    .loading-mask,
    .loading-mask .loader>img {
        bottom: 0;
        left: 0;
        margin: auto;
        position: fixed;
        right: 0;
        top: 0;
        z-index: 9900;
    }
}
```


###### Using LoaderLayerComponent

1. Put the tag to your template

```html
<app-loaderlayer></app-loaderlayer>
```

2. Send show message to loaderlayer in your component to show loader layer

```typescript
  this.mq.notify('app-loaderlayer', {type:'show', data:null})
```

3. Send hide message to loaderlayer in your component to hide loader layer

```typescript
  this.mq.notify('app-loaderlayer', {type:'hide', data:null})
```