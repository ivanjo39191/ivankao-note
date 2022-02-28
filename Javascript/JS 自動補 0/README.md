# JS 自動補 0

Created: February 1, 2022 5:22 PM
Tags: Javascript

```jsx
/* 自動補0 */
function paddingLeft(str,lenght){
  if(str.length >= lenght){
    return str;
  }
  else {
    return paddingLeft("0" +str,lenght);
  }
}
```

Refenerces

[https://dotblogs.com.tw/aquarius6913/2011/05/10/24655](https://dotblogs.com.tw/aquarius6913/2011/05/10/24655)