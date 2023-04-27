# CustomMoviePlayer
動画再生をカスタマイズしたい

expo-avのインストール
権限の設定が必要になる
[expo/packages/expo-av at main · expo/expo](https://github.com/expo/expo/tree/sdk-47/packages/expo-av#installation-in-bare-react-native-projects)を参考にapp.config.jsに以下を追記した。
追記後に`npm run prebuild`をして（ネイティブ・コードの設定ファイルが作り直される）からビルドする。
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

実装したこと
- 進捗率を動画から撮ってきた再生位置と動画の長さから算出してバー表示にする
- reanimatedで進捗バーを作った。画面の動画情報は直接反映しないので、動画の長さが決まっているかつ停止しないならこれでOK。
- Skipボタンで動画を止めて画面遷移する
- 

```typescript
import {FontAwesome} from '@expo/vector-icons';
import {useNavigation} from '@react-navigation/native';
import {Video, ResizeMode, AVPlaybackStatus, AVPlaybackStatusSuccess} from 'expo-av';
import React, {useCallback, useEffect, useRef, useState} from 'react';
import {View, StyleSheet} from 'react-native';
import {Button, Text} from 'react-native-elements';
import {State} from 'react-native-gesture-handler';
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
  const navigation = useNavigation();
  const onGoToInstructionButtonPress = useCallback(async () => {
    await video.current?.pauseAsync(); // 次のページに行く前に動画を止める
    navigation.navigate('Instructions');
  }, [navigation]);

  const video = useRef<Video>(null);
  const [status, setStatus] = useState<AVPlaybackStatusSuccess | null>(null);
  const [progress, setProgress] = useState<number>(0);

  const barWidth = useSharedValue(0);
  const animatedStyle = useAnimatedStyle(
    () => ({
      width: barWidth.value,
    }),
    [],
  );

  const startProgress = useCallback(() => {
    if (barWidth.value === 0) {
      barWidth.value = withTiming(300, {duration: 30000, easing: Easing.linear});
    } else {
      barWidth.value = 0;
    }

    console.log('aaaaa');
  }, [barWidth]);

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

  const updatePlaybackuseCallback = useCallback((status: AVPlaybackStatus) => {
    if (status.isLoaded && status.durationMillis) {
      const progress = status.positionMillis / status.durationMillis;
      setStatus(status);
      setProgress(progress);
    }
  }, []);

  return (
    <SafeAreaView style={styles.container}>
      <Text>aaaaa</Text>
      <Text>{progress}</Text>
      <Video
        ref={video}
        style={styles.video}
        useNativeControls={false} // falseでコントローラが消える
        resizeMode={ResizeMode.CONTAIN}
        isLooping
        progressUpdateIntervalMillis={1000}
        onPlaybackStatusUpdate={updatePlaybackuseCallback}
      />
      <View style={{position: 'absolute', top: 300}}>
        <View style={{backgroundColor: 'grey', height: 5, width: 300}}>
          {/* レンダリングの間隔があるのでカクカクしてしまう */}
          <View style={{backgroundColor: 'black', height: 5, width: 300 * progress}} />
        </View>
      </View>
      <View style={{height: 12}} />

      <View style={styles.buttons}>
        {status && (
          <Button
            title={status.isPlaying ? 'Pause' : 'Play'}
            onPress={() => (status.isPlaying ? video.current?.pauseAsync() : video.current?.playAsync())}
          />
        )}
      </View>
      <View style={{height: 24}} />
      <View>
        <View style={{backgroundColor: 'grey', height: 5, width: 300}}>
          {/* 動画の長さが固定で途中停止しないならreanimatedで進捗率のバーを作った方がなめらかに表示できる */}
          <Animated.View style={[styles.bar, animatedStyle]} />
        </View>
        <View style={{height: 12}} />
        <Button title="Animation start / reset" onPress={startProgress} />
      </View>
      <Text>bbb</Text>
      <Button title="Skip Video" onPress={onGoToInstructionButtonPress} />
    </SafeAreaView>
  );
};
const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
  bar: {
    backgroundColor: 'black',
    height: 5,
    width: 10,
  },
  video: {height: 500},
  buttons: {},
});

```
