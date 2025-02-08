---
Status: In progress
tags:
  - bread
  - frontend
---
jest: [https://jestjs.io/docs/configuration](https://jestjs.io/docs/configuration#modulenamemapper-objectstring-string--arraystring)

playwright [https://habr.com/ru/articles/597293/](https://habr.com/ru/articles/597293/)

vitest [https://vitest.dev/guide/](https://vitest.dev/guide/)

```JSON
// module name resolution in webpack
// this thing in config
{ 
	"^myModule(.*)$": "<rootDir>/src/components/react$1"
}
// will do this: myModule/SOMETHING => <rootDir>/src/components/react/SOMETHING"
```

konva-jest [https://github.com/konvajs/konva#4-nodejs-env](https://github.com/konvajs/konva#4-nodejs-env)