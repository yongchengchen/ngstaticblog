<!--
Categories = ["Development", "Angular"]
Description = ""
Tags = ["Development", "Angular"]
date = "2016-10-26T21:47:31-08:00"
title = "Angular 2/4 Global Variable Service"
-->

## Angular 2/4 Global Variable Service

#### Why doing this?

When I tried to use Angular 2/4 combining Magento 2 REST API to build our own Sales/Orders panel, I need to pass some Magento2 variables to Angular.

Magento 2 using Kockout.js to render UI but the Sales/Orders panel is not very efficiency. So we just use Angular 4 to quickly implement a new panel which is a all in one panel.

#### Implement

Here's the source code of GobalVariableService.

```typescript
import { Injectable } from '@angular/core';
import { ElementRef } from '@angular/core';

interface VAR_LIST {
    [key:string]:string;
}

@Injectable()
export class GobalVariableService {
    private list:VAR_LIST;

    constructor() {
        this.list = {};
        console.log('GobalVariableService constructor')
    }

    get(key:string):string {
        if (this.list[key]) {
            return <string>this.list[key];
        }
        return '';
    }
    
    set(key:string, value:any) {
        this.list[key] = value;
    }

    initFromRootElement(rootElement:ElementRef) {
        let attr:string = rootElement.nativeElement.getAttribute('globals');
        let items =  attr.split(';');
        let globals = {};
        for(let item of items) {
            let values = item.split('=');
            this.set(values[0], values[1]);
        }
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
import { GobalVariableService } from './global.variable.service';

@NgModule({
    declarations: [
        AppComponent
    ],
    imports: [
        BrowserModule,
        AppRoutingModule
    ],
    providers: [
        GobalVariableService
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

##### 2. Inject it from your root component(AppComponent) of your Angular project

It depends on ElementRef, so you need to inject ElementRef as well.

And you call the member initFromRootElement to init global variables.

```typescript
import { Component, ElementRef } from '@angular/core';
import { GobalVariableService } from './global.variable.service';

@Component({
    selector: 'app-root',
    templateUrl: './app.component.html',
    styleUrls: ['./app.component.scss']
})
export class AppComponent {
    constructor(private elementRef:ElementRef, private gvs:GlobalVarService) {
        this.gvs.initFromRootElement(elementRef);
    }
}
```

##### 3. Prepare global variables in you main.html


Please combine global variables as: "key1=value1;key2=value2";

And the attribute name is globals for root component selector.

```html
<app-root globals="var1=test1;var2=test2">
```

##### 4. Retrieve global variable in components

```typescript
import { Component } from '@angular/core';
import { GobalVariableService } from './global.variable.service';

@Component({
  selector: 'app-globalvariable-user',
  templateUrl: './globalvariable.user.component.html',
  styleUrls: ['./globalvariable.user.component.scss']
})
export class GlobalVariableUserComponent {
  constructor(private gvs:GobalVariableService) { 
    console.log(this.gvs.get('var1'))   //it will output test1
  }
}
```

