# eslint 规则禁用

## 单文件禁用

```js
/* eslint-disable */
```

## 单文件规则禁用

```js
/* eslint-disable no-alert, no-undef */
```

## 单行禁用

```js
// eslint-disable-next-line

// eslint-disable-line
```

## 单行禁用规定规则

```js
// eslint-disable-next-line no-alert, no-undef

// eslint-disable-line no-alert, no-undef
```

## 禁用某个插件的某个规则

```js
/* eslint-disable airbnb/no-console */

// eslint-disable-next-line airbnb/no-console

// eslint-disable-line airbnb/no-console* v ***

```

