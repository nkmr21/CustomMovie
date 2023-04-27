# CustomMoviePlayer
動画再生をカスタマイズしたい

expo-avのインストール
権限の設定が必要になる
[expo/packages/expo-av at main · expo/expo](https://github.com/expo/expo/tree/sdk-47/packages/expo-av#installation-in-bare-react-native-projects)を参考にapp.config.jsに以下を追記した。
追記後に`npm run prebuild`をしてからビルドする。
```javascript
android: {
      permissions: ['RECORD_AUDIO'],
},
plugins: [
      [
        'expo-av',
        {
          microphonePermission: 'Allow $(PRODUCT_NAME) to access your microphone.',
        },
      ],
 ]    
```


動画についてassetsからのソース読み込みは公式にexampleがなかったので、手探りで実装した。
[ここ](https://docs.expo.dev/versions/latest/sdk/video/#source)にrefでloadAsyncしてねという雑な説明だけがある。

```typescript

import {FontAwesome} from '@expo/vector-icons';
import {Video, ResizeMode} from 'expo-av';
import React, {useCallback, useEffect, useRef, useState} from 'react';
import {View, StyleSheet} from 'react-native';
import {Text} from 'react-native-elements';
import Animated, {
  useAnimatedStyle,
  useSharedValue,
  withTiming,
  Easing,
  withRepeat,
  withSequence,
  withDelay,
} from 'react-native-reanimated';
import {SafeAreaView} from 'react-native-safe-area-context';
import {
  VictoryBar,
  VictoryAxis,
  VictoryPolarAxis,
  VictoryGroup,
  VictoryArea,
  VictoryChart,
  VictoryTheme,
  Arc,
  Line,
} from 'victory-native';

export const Home: React.FC = () => {
  const video = useRef<Video>(null);
  const [status, setStatus] = React.useState(undefined);
  // const {sound: playbackObject} = await video.loadAsync({uri: require('assets/big_buck_bunny.mp4')});
  useEffect(() => {
    // const source = require('../../../assets/big_buck_bunny.mp4');
    // video.current?.loadAsync(require('../../../assets/big_buck_bunny.mp4')).catch(e => {
    //   console.log(e);
    // });
    video.current?.loadAsync(require('../../../assets/D0002060618_00000_V_000.mp4')).catch(e => {
      console.log(e);
    });
  }, []);

  return (
    <SafeAreaView style={styles.container}>
      <Text>aaaaa</Text>
      <Video
        ref={video}
        style={styles.video}
        // source={{
        //   // uri: 'https://d23dyxeqlo5psv.cloudfront.net/big_buck_bunny.mp4',//見れた
        //   uri: require('src/assets/big_buck_bunny.mp4'),
        // }}
        useNativeControls
        resizeMode={ResizeMode.CONTAIN}
        isLooping
        // onPlaybackStatusUpdate={status => setStatus(() => status)}
      />
      {/* <View style={styles.buttons}>
        {status && (
          <Button
            title={status.isPlaying ? 'Pause' : 'Play'}
            onPress={() => (status.isPlaying ? video.current.pauseAsync() : video.current.playAsync())}
          />
        )}
      </View> */}
      <Text>bbb</Text>
    </SafeAreaView>
  );
};
const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  video: {height: 500},
  buttons: {},
});

```
