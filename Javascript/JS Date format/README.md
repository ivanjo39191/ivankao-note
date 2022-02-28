# JS Date format

Created: February 1, 2022 5:22 PM
Tags: Javascript

```jsx
/* 將日期時間格式格式化為不含秒數的時間 */
export function splitTime(string) {
  let hours = paddingLeft(String(new Date(string).getHours()), 2);
  let minutes = paddingLeft(String(new Date(string).getMinutes()), 2);
  const time_res = hours + ':' + minutes
  return time_res
}
```

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

References

[https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)

[https://stackoverflow.com/questions/14638018/current-time-formatting-with-javascript](https://stackoverflow.com/questions/14638018/current-time-formatting-with-javascript)

[https://codertw.com/前端開發/254123/](https://codertw.com/%E5%89%8D%E7%AB%AF%E9%96%8B%E7%99%BC/254123/)