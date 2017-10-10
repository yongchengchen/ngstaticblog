<!-- 
Categories = ["Development", "Angular"]
title = "Angular 2/4 Async processor delegator component"
Description = ""
Tags = ["Development", "Angular 2/4"]
date = "2017-10-09T10:38:31+11:00"
-->
# An Angular 2/4 Async processor delegator component

#### Why doing this?

Sometimes we need to run a long time backend process and it always runs overtime.
And the better way is we separate the long time process into many small processes.
Here's a component to delegate these small processes and finish a big process.

It supports auto start, pause, restart when processing your request.

#### Implement

Here's the source code of GobalVariableService.

##### Dependcies

* MQService
* uuid
* NgbModal, NgbModalRef, ngb-progressbar from ng-bootstrap

###### 1. async.processor.component.ts
```typescript
import { Component, Input, OnInit, OnDestroy, ViewChild, ElementRef } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Subscription } from 'rxjs/Subscription';
import { NgbModal, NgbModalRef } from '@ng-bootstrap/ng-bootstrap';
import { MQService } from './mq.service';
import * as uuid from "uuid";

export interface ASYNC_TASK_ACTION_BUILDER_RETURN {
    url:string,
    body:any,
    options:any
}

export interface ASYNC_TASK_REPOSITORY_CALLBACK_RETURN {
    to_continue:boolean, 
    items:string[],
    total:number
}

//if support pagination, repository's url should have reserve place({0}) for pagination
export interface ASYNC_TASK_PARAMETER {
    action_builder(ids:string[],uuid:string):ASYNC_TASK_ACTION_BUILDER_RETURN;
    _token?:string;
    step?:number;
    autostart?:boolean;
    keys?:string[];
    repository?:{
        url:string,
        callback(resp:any):ASYNC_TASK_REPOSITORY_CALLBACK_RETURN,
        pagination?:boolean
    }
}

interface ANALYSIS_PROCESS_RESULT {
    success:boolean,
    messages:{[key:string]:string};
}

interface CONSOLE_LOG {
    level:string,
    info:string
}

@Component({
    selector: 'async-processer-delegator',
    templateUrl: './async.processor.component.html',
    styleUrls: ['./async.processor.component.scss'],
    providers: [NgbModal]
})
export class AsyncProcessorComponent implements OnInit, OnDestroy{
    @ViewChild('content') content:ElementRef;
    private mqSubs: Subscription;
    private modal:NgbModalRef;
    private pending:boolean = false;
  
    private process_ids:string[];
    private error_count:number = 0;
    private parameter:ASYNC_TASK_PARAMETER = null
    private _uuid:string = '';
  
    i:number = 0;
    total:number = 0;
    buttonTitle:string = 'Start'
    bCanStart:boolean = false;
    bCanRetry:boolean = false;
    bCanClose:boolean = false;
    bCanLoadResult:boolean = false;
    logs:CONSOLE_LOG[] = [];
    
    private taskFrom:string = '';  //who send task message to the module

    constructor(private modalService: NgbModal, private http:HttpClient, private mq:MQService) { }

    show() {
        this.modal = this.modalService.open(this.content,{ backdrop : 'static', keyboard  : false});
    }

    ngOnInit() {
        this.mqSubs = this.mq.listen('async-processer-delegator').subscribe(msg => {
            if (msg.type == 'async-process') {
                this.taskFrom = msg.from;
                this.attachTask(msg.load);
                this.show();
            }
        });
    }

    ngOnDestroy() {
        this.subscription.unsubscribe();
    }

    attachTask(parameter:ASYNC_TASK_PARAMETER) {
        this.pending = true;
        this.logs = [];
        this.i = 0;
        this.error_count = 0;
        this.parameter = parameter;
        this.bCanStart = true;
        this.buttonTitle = 'Start'
        this.bCanRetry = false;
        this.bCanClose = false;
        this.bCanLoadResult = false;

        if (!this.parameter.step) {
            this.parameter.step = 1;
        }
        if (this.parameter.keys != undefined) {
            this.process_ids = this.parameter.keys;
            if (this.parameter.autostart) {
                this.start();
            }
            return;
        } else {
            if (this.parameter.repository === undefined) {
                this.log('error', 'Parameter should has keys or repository');
                this.bCanClose = true;
                return;
            }
            this.process_ids = [];
            this.retrieveTaskResource(1);
        }
    }

    private retrieveTaskResource(page:number) {
        let url = this.parameter.repository.url;
        if (this.parameter.repository.pagination) {
            url = url.replace('{0}', String(page));
        }

        this.http.get(url).subscribe(resp => {
            if (this.parameter.repository.callback) {
                let ret = this.parameter.repository.callback(resp);
                this.process_ids = this.process_ids.concat(ret.items);
                if (ret.to_continue && this.parameter.repository.pagination) {
                    this.retrieveTaskResource(++page);
                }
                this.total = ret.total;
                if (page<=2) {
                    this.start();
                }
                console.log('return value', this.process_ids);
            }
            
        });
    }

    start() {
        this._uuid = uuid.v4();
        this.pending = false;
        this.buttonTitle = 'Pause'
        this.log('system', '>>>>>>>>>start processing>>>>>>>>>');
        this.tryprocess();
    }

    togglePauseOrResume() {
        this.pending = !this.pending;
        if (this.pending) {
            this.buttonTitle = 'Resume'
            this.log("warn", "|||||||||||||||   Process paused   |||||||||||||||")
        } else {
            this.buttonTitle = 'Pause'
            this.log('warn', '>>>>>>>>>Process resumed>>>>>>>>>');
            this.tryprocess();
        }
    }
    
    retry() {
        this.pending = false;
        this.i = 0;
        this.error_count = 0;
        this.bCanStart = true;
        this.bCanRetry = false;
        this.bCanClose = false;
        this.bCanLoadResult = false;
        this.start();
    }

    loadResult() {
        this.mq.notify(this.taskFrom, {type:'async_process_finished', load:{ uuid:this._uuid}});
        this.modal.close();
    }

    log(level:string, message:string) {
        this.logs.unshift({level:level,info:message});
    };

    tryprocess() {
        if (!this.pending) {
            let post_keys = [];
            for(let i=0; i<this.parameter.step; i++) {
                if (this.i < this.process_ids.length) {
                    post_keys.push(this.process_ids[this.i]);
                    this.i++;
                }
            }
            if (post_keys.length > 0) {
                this.process({
                    keys: post_keys,
                    _token: this.parameter._token
                }, 'post');
            } else {
                this.log('system', '========= process finished =========');
                if (this.error_count > 0) {
                    this.log('error', '|||||||||||||||||||||||||||||||||||||||||||||');
                    this.log('error', '===== You got ' + this.error_count +' errors, please fix first ====');
                    this.log('error', '|||||||||||||||||||||||||||||||||||||||||||||');
                    this.bCanRetry = true;
                    this.bCanClose = true;
                    this.bCanLoadResult = true;
                } else {
                    this.loadResult();
                }
                this.bCanStart = false;
            }
        }
    }

    process(params, httpmethod) {
        let action = this.parameter.action_builder(params.keys, this._uuid);
        this.http.post(action.url, action.body, action.options).subscribe(resp => {
            for(let key in resp['messages']) {
                this.log(resp['messages'][key],key);
                if (resp['messages'][key] =='error') {
                    this.error_count++;
                }
            }
            this.tryprocess();
        });
    }
}
```
###### 2. async.processor.component.html
```typescript
<ng-template #content let-c="close" let-d="dismiss">
    <div class="modal-header">
        <h3 class="modal-title">Async Task Console</h3>
    </div>
    <div class="modal-body common-modal-dialog">
        <div class="processor-container">
            <p>
                <ngb-progressbar type="success" [value]="i" [striped]="true" [animated]="true" [max]="total"></ngb-progressbar>
            </p>
            <div class="progress">Process <b>{{i}} of {{total}}</b></div>
        </div>

        <div>
            <button class="btn btn-info" *ngIf="bCanStart" (click)="togglePauseOrResume()">{{buttonTitle}}</button>
            <button class="btn btn-info" *ngIf="bCanRetry" (click)="retry()">Retry</button>
            <button class="btn btn-info" *ngIf="bCanLoadResult" (click)="loadResult()">Load Result Anyway</button>
        </div>
        <br/>

        <div class='processor-console'>
            <span *ngFor="let item of logs" class="console {{item.level}}">{{ item.info }}</span>
        </div>
    </div>
    <div class="modal-footer close_btn">
        <button class="btn btn-warning" type="button" data-dismiss="modal" *ngIf="bCanClose">Close</button>
    </div>

</ng-template>
```

###### 3. async.processor.component.scss
```typescript
.processor-container {
    width: 100%;
    overflow-y: scroll;
}

div.progress {
    margin-top: -16px;
    text-align: center;
    display: block !important;
}

.processor-console {
    width: 100%;
    max-height: 800px;
    overflow-y: scroll;
}

.error {
    color: red !important;
}

.info {
    color: lime !important;
}

.warn {
    color: yellow !important;
}

.system {
    color: #ff3388 !important;
}

.console {
    font-size: 14px;
    background: #000;
    color: #ccc;
    display: block;
    padding: 20px 0 0 5px;
    margin: -20px 0 0 0;
}
```

#### How to use
Here's a simple example to use 


##### Step1. Declare it in the user module
* example.module.ts
```typescript
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HttpClientModule } from '@angular/common/http';
import { NgbModule, NgbDropdownModule } from '@ng-bootstrap/ng-bootstrap';

import { ExampleComponent } from './example.component';

import {  AsyncProcessorComponent } from './async.processor.component';

@NgModule({
  imports: [
    CommonModule,
    NgbDropdownModule,
    NgbModule.forRoot(),
    OrderReportRoutingModule,
    HttpClientModule
  ],
  declarations: [ExampleComponent, AsyncProcessorComponent],
})
export class ExampleModule { }
```


##### Step2. Put tag to user component html

* example.component.html

```html
<async-processer-delegator></async-processer-delegator>
<button (click)="process()"></button>
```

##### Step3. Prepare in component

* example.component.ts

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { MQService } from '@ngxsuit';
import { HttpClient } from '@angular/common/http';
import { Subscription } from 'rxjs/Subscription';

import { ASYNC_TASK_ACTION_BUILDER_RETURN, 
    ASYNC_TASK_REPOSITORY_CALLBACK_RETURN, 
    ASYNC_TASK_PARAMETER } from './async.processor.component'

@Component({
  selector: 'async-example',
  templateUrl: './example.component.html',
  styleUrls: ['./example.component.scss']
})
export class ExampleComponent implements OnInit, OnDestroy{
  private mqSubs:Subscription;
  private _uuid:string = '';

  constructor(private mq:MQService, private http:HttpClient) { }
  
  ngOnInit() {
    this.mqSubs = this.mq.listen('async-example').subscribe(msg => {
      if (msg.type = 'async_process_finished') {
        if (msg.load.uuid != undefined) {
          this._uuid = msg.load.uuid;
          
        }
        //Add code to handle when aysnc process is done.
      }
    });
  }

  ngOnDestroy() {
    this.mqSubs.unsubscribe();
  }

  process() {
    let backendUrl2getAllProcessesKeys = 'this url returns process identities';
    let parameters = {
      action_builder:this.action_builder,
      _token:'',
      step:10,
      autostart:true,
      repository:{url:backendUrl2getAllProcessesKeys, pagination:true, callback:this.repository_callback}
    }
    this.mq.notify('async-processer-delegator', {from:'async-example', type:"async-process", load:parameters});
  }

  //this function define how to build a sub process url parameters
  action_builder(ids:string[], uuid:string):ASYNC_TASK_ACTION_BUILDER_RETURN {
    let subprocessurl = 'your backend url which can process sub process';
    //uuid is the key to combine your whole large process.
    let body = {ids:ids.join(','), uuid:uuid}
    return {url:subprocessurl, body:body, options:{}};
  }
  
  //this function defines to allow you fetch sub process keys by pagination, 
  // so you don't need to down load a full sub process keys at one time.
  // And it need your backend to support pagination
  // It does 3 things
  // 1. to determine if need to request an other pagination to your server
  // 2. to get response sub process keys
  // 3. to get total sub process amount 
  repository_callback(resp):ASYNC_TASK_REPOSITORY_CALLBACK_RETURN {
    let to_continue = false;
    
    //parse resp here, and check if 

    let items;       // get from resp
    let total = 0;   // get total from resp

    //These logic bases on your server side logic.

    return {to_continue:to_continue, items: items, total:total};
  }
}
```