# Two families of targets

| | Native React Native | Web build |
| --- | --- | --- |
| Targets | Android TV (incl. Google TV, Fire TV), Apple TV (tvOS) | LG (webOS), Samsung (Tizen) |
| Runtime | React Native (the `react-native-tvos` fork) | Browser engine on the TV, via `react-native-web` |
| Bundler | **Metro** | **webpack** |
| Focus engine | Native platform focus (Android focus search, tvOS focus engine) | None built in. A spatial navigation library (Norigin) handles it |
| Packaging | APK/AAB, Xcode archive | `.ipk` (webOS), `.wgt` (Tizen) |

The same React components can serve both families, but **focus, keys, and Back work differently** in each. Code that touches those areas usually needs a platform split (`Platform.OS === 'web'`, or `.web.tsx` / `.native.tsx` files). See [Focus on the web build](./03-focus-on-web.md).