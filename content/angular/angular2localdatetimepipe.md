<!--
Categories = ["Development", "Angular"]
Description = ""
Tags = ["Development", "Angular"]
date = "2016-10-26T21:47:31-08:00"
title = "Angular 4 UTC time to Local time Pipe"
-->
# Angular 4 UTC time to Local time Pipe

* Why doing this?

Our REST api is served for many different time zone and it's datetime is loaded by UTC time. And angular default date pipe doesn't support datetime conversion. 

* Implement

The Pipe extends Angular default DatePipe, and reuse its transform function to get correct datetime string. Then convert it back to UTC Date object by adding GMT+00 (UTC), then use toLocaleString to convert to local time zone. 

```typescript
import { Pipe } from '@angular/core';
import { DatePipe } from '@angular/common';

@Pipe({name: 'localdate'})
export class LocalDatePipe extends DatePipe {
  transform(value: any, pattern: string = 'Australia/Sydney'): string|null {
    let datestr = super.transform(value, 'yyyy-MM-dd hh:mm:ss');
    if (datestr == null) {
      return null;
    }
    datestr = datestr + ' GMT+0000';
    let date = new Date(datestr);
    return date.toLocaleString('en-au', {timeZone:pattern});
  }
}
```

#### How To Use
```html
date_expression | localdate[:timezone]
```

####Examples
```html
{{ dateObj | localdate }}           // convert to Australia/Sydney timezone
{{ dateObj | date:'Asia/Beijing' }} // convert to Asia/Beijing timezone
```
