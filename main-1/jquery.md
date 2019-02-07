# JQuery tips

Note: you need to install the `jquery.d.ts` file for these tips

## Quickly define a new plugin

Just create `jquery-foo.d.ts` with:

```typescript
interface JQuery {
  foo: any;
}
```

And now you can use `$('something').foo({whateverYouWant:'hello jquery plugin'})`

