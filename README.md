
A dart binding of [quickjs](https://bellard.org/quickjs/) with latest `native-assets`.

Inspired by [flutter_js](https://pub.dev/packages/flutter_js), quickjs-dart is also used to run javascript code as a native citzen, but without flutter channels, and thanks to native_assets_cli, quickjs-dart could be integrated into any dart app (not flutter currently) with all platform support(except web). The integrated quickjs version is `2025-04-26`.

Have a try with `dart --enable-experiment=native-assets run example/example.dart`

## Note

1. Currently, `native-assets` has moved to dart dev channel, so you would have to replace dart sdk with `3.10.0-14.0.dev`, which is working fine now.
2. `quickjs` shared library was compiled in `ubuntu:18.04` container, it is compatible with `GLIBC_2.27`.
3. `dart run --define=YOUR_ENV=env_value -DYOUR_ENV=env_value` seems not work with `3.10.0-14.0.dev`.

## asynchronous

All interops with quickjs native code through `dart:ffi` run in a separated isolate, this could make sure js evaluation would not block the main isolate, also a different point from flutter_js.
```dart
final manager = await JsEngineManager.create();
final engine = await manager.createEngine('my-tag');
final result = await engine.eval('console.log("Hello~");');
print(result.stdout); // "Hello~"
await engine.dispose();
await manager.dispose();
```

Of course, quickjs-dart could aslo run in main isolate synchronously, just using different objects:
```dart
final manager = NativeEngineManager();
final engine = NativeJsEngine(name: 'my-tag');
final result = engine.eval('3-4');
print(result.value); // "-1"
engine.dispose();
manager.dispose();
```

## notify callback

If you want to receive data directly from js code, you need register a notify callback in Dart side:
```dart
const code = """
let scope = {
  send(obj) {
    _ffiNotify("_sendMsg", JSON.stringify(obj));
  }
};
scope.send({"key": $variable_in_dart});
""";
engine.eval(code); // receive message but do nothing.
engine.registerBridge('_sendMsg', (obj) {
  final val = obj['key']; // $variable_in_dart
});
engine.eval(code); // receive message and call the callback in dart world.
```
remember to use the builtin `_ffiNotify` in js world.

## builtin js functions

- console.log
- setTimeout
- \_ffiNotify
