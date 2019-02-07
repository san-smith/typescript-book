# Начало работы

* [Начало работы с TypeScript](./#getting-started-with-typescript)
* [Версия TypeScript](./#typescript-version)

## Начало работы с TypeScript

TypeScript компилируется в JavaScript. JavaScript это то, что вы на самом деле собираетесь выполнить \(в браузере или на сервере\). Итак, вам понадобится следующее:

* Компилятор TypeScript \(OSS доступен [в исходниках](https://github.com/Microsoft/TypeScript/) и в [NPM](https://www.npmjs.com/package/typescript)\)
* Редактор кода TypeScript \(вы можете использовать блокнот, если хотите, но я использую [vscode 🌹](https://code.visualstudio.com/) с [расширением](https://marketplace.visualstudio.com/items?itemName=basarat.god), которое я написал. Также есть множество других [IDE с поддержкой TypeScript](https://github.com/Microsoft/TypeScript/wiki/TypeScript-Editor-Support)\)

### Версия TypeScript

Вместо использования _стабильной_ версии компилятора TypeScript мы будем использовать ночную сборку и покажем много новых вещей в этой книге, которые могут быть ещё не включены в стабильную версию. Я обычно рекомендую людям использовать ночные сборки, потому что ночная сборка \*\*обрабатывает больше ошибок чем стабильный релиз\*\*..

Вы можете установить её через командную строку, выполнив

```text
npm install -g typescript@next
```

И теперь в командной строке `tsc` будет означать последнюю версию компилятора. Многие IDE также поддерживают это. В частности:

* Вы можете попросить vscode использовать эту версию путем создания файла `.vscode/settings.json` со следующим содержанием:

```javascript
{
  "typescript.tsdk": "./node_modules/typescript/lib"
}
```

### Получение исходного кода

Исходный код для этой книги доступен в репозитории книги на Гитхаб: [https://github.com/basarat/typescript-book/tree/master/code](https://github.com/basarat/typescript-book/tree/master/code). Большинство примеров кода могут быть скопированы в vscode, и вы можете играть с ними как есть. Для примеров кода, которые требуют дополнительной настройки \(например, модули npm\), мы дадим ссылку на пример кода перед представлением кода, например:

`this/will/be/the/link/to/the/code.ts`

```typescript
// This will be the code under discussion
```

Оставим в стороне настроку среды и перейдем к синтаксису TypeScript.

