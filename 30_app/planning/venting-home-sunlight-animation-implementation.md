# 속마음 홈 숲 햇빛 애니메이션 구현안

## 목적

속마음 메인 숲 화면에 따뜻한 금빛 햇빛이 천천히 흔들리는 효과를 추가한다.
첫 배포에서는 새 의존성이나 복잡한 그래픽 엔진 없이 확실하게 동작하는 최소
버전을 우선한다.

## 결론

1차 구현은 기존에 설치된 `expo-linear-gradient`와
`react-native-reanimated`만 사용한다.

- 넓고 부드러운 `LinearGradient` 3개를 햇빛 줄기로 배치한다.
- 개별 광선이 아니라 광선을 담은 그룹 전체만 천천히 움직인다.
- 숲 배경 위, 돌멩이·버튼·텍스트 아래에 배치한다.
- Lottie, Skia, 파티클, 실시간 블러는 1차 범위에서 제외한다.
- iOS와 Android에서 동일한 기본 알파 합성만 사용한다.

현재 앱에는 다음 패키지가 이미 설치되어 있다.

- `expo-linear-gradient`: `~56.0.4`
- `react-native-reanimated`: `4.3.1`
- `lottie-react-native`: `~7.3.4`

Lottie도 설치되어 있지만, 이번 효과는 별도 JSON 애니메이션 에셋 없이 구현할 수
있으므로 최소 버전에서는 사용하지 않는다.

## 1차 구현 범위

### 동작 사양

| 항목 | 기본값 | 권장 조정 범위 |
| --- | ---: | ---: |
| 왕복 시간 | 8초 | 8~12초 |
| 좌우 이동 거리 | ±12px | ±8~16px |
| 그룹 회전 | ±0.6도 | ±0.4~0.8도 |
| 그룹 불투명도 | 0.48~0.62 | 최대 0.7 이하 |
| 가장 밝은 광선 색 | `rgba(255, 241, 184, 0.28)` | 알파 0.20~0.32 |

배경 자체가 밝기 때문에 순백색은 사용하지 않는다. 가장 밝은 색의 알파가 `0.4`를
넘으면 광선이 흰색처럼 보일 가능성이 높다.

### 레이어 순서

1. `VentingSceneBackground`
2. `VentingSunlightOverlay`
3. 기존 `Container`
4. 돌멩이, 버튼, 텍스트

`VentingSunlightOverlay`는 `pointerEvents="none"`으로 설정해 기존 터치와
제스처를 방해하지 않는다.

## 구현 코드

### 새 파일

경로:

`src/screens/venting/home/VentingSunlightOverlay.tsx`

```tsx
import {useIsFocused} from '@react-navigation/native';
import {LinearGradient} from 'expo-linear-gradient';
import {useEffect} from 'react';
import {StyleSheet} from 'react-native';
import Animated, {
  cancelAnimation,
  Easing,
  interpolate,
  ReduceMotion,
  useAnimatedStyle,
  useReducedMotion,
  useSharedValue,
  withRepeat,
  withTiming,
} from 'react-native-reanimated';

const SUNLIGHT_DURATION_MS = 8000;
const SUNLIGHT_TRANSLATE_X = 12;
const SUNLIGHT_ROTATION_DEG = 0.6;

const SUNLIGHT_COLORS = [
  'rgba(255, 235, 167, 0)',
  'rgba(255, 232, 154, 0.12)',
  'rgba(255, 241, 184, 0.28)',
  'rgba(246, 217, 120, 0.1)',
  'rgba(239, 202, 92, 0)',
] as const;

const SUNLIGHT_LOCATIONS = [0, 0.25, 0.51, 0.76, 1] as const;

export function VentingSunlightOverlay() {
  const isFocused = useIsFocused();
  const reduceMotion = useReducedMotion();
  const progress = useSharedValue(0.5);

  useEffect(() => {
    cancelAnimation(progress);

    if (!isFocused || reduceMotion) {
      progress.value = 0.5;
      return;
    }

    progress.value = 0;

    progress.value = withRepeat(
      withTiming(1, {
        duration: SUNLIGHT_DURATION_MS,
        easing: Easing.inOut(Easing.cubic),
        reduceMotion: ReduceMotion.System,
      }),
      -1,
      true,
      undefined,
      ReduceMotion.System,
    );

    return () => {
      cancelAnimation(progress);
    };
  }, [isFocused, progress, reduceMotion]);

  const animatedStyle = useAnimatedStyle(() => {
    const translateX = interpolate(
      progress.value,
      [0, 1],
      [-SUNLIGHT_TRANSLATE_X, SUNLIGHT_TRANSLATE_X],
    );

    const rotation = interpolate(
      progress.value,
      [0, 1],
      [-SUNLIGHT_ROTATION_DEG, SUNLIGHT_ROTATION_DEG],
    );

    const opacity = interpolate(
      progress.value,
      [0, 0.5, 1],
      [0.48, 0.62, 0.52],
    );

    return {
      opacity,
      transform: [{translateX}, {rotateZ: `${rotation}deg`}],
    };
  });

  return (
    <Animated.View
      accessible={false}
      accessibilityElementsHidden
      importantForAccessibility="no-hide-descendants"
      pointerEvents="none"
      style={styles.overlay}>
      <Animated.View style={[styles.movingGroup, animatedStyle]}>
        <LinearGradient
          colors={SUNLIGHT_COLORS}
          locations={SUNLIGHT_LOCATIONS}
          start={{x: 0, y: 0.5}}
          end={{x: 1, y: 0.5}}
          style={[styles.ray, styles.rayOne]}
        />

        <LinearGradient
          colors={SUNLIGHT_COLORS}
          locations={SUNLIGHT_LOCATIONS}
          start={{x: 0, y: 0.5}}
          end={{x: 1, y: 0.5}}
          style={[styles.ray, styles.rayTwo]}
        />

        <LinearGradient
          colors={SUNLIGHT_COLORS}
          locations={SUNLIGHT_LOCATIONS}
          start={{x: 0, y: 0.5}}
          end={{x: 1, y: 0.5}}
          style={[styles.ray, styles.rayThree]}
        />
      </Animated.View>
    </Animated.View>
  );
}

const styles = StyleSheet.create({
  overlay: {
    ...StyleSheet.absoluteFillObject,
    overflow: 'hidden',
  },

  /**
   * 광선이 회전할 때 위·아래 끝이 화면 안에 나타나지 않도록
   * 실제 화면보다 크게 만든다.
   */
  movingGroup: {
    position: 'absolute',
    top: -160,
    right: -90,
    bottom: -220,
    left: -90,
  },

  ray: {
    position: 'absolute',
    top: 0,
    bottom: 0,
  },

  rayOne: {
    left: '34%',
    width: '24%',
    opacity: 1,
    transform: [{rotateZ: '11deg'}],
  },

  rayTwo: {
    left: '56%',
    width: '17%',
    opacity: 0.78,
    transform: [{rotateZ: '8deg'}],
  },

  rayThree: {
    left: '74%',
    width: '13%',
    opacity: 0.58,
    transform: [{rotateZ: '6deg'}],
  },
});
```

### 메인 화면에 삽입

대상:

`src/screens/venting/VentingHomeScreen.tsx`

import를 추가한다.

```tsx
import {VentingSunlightOverlay} from '~/screens/venting/home/VentingSunlightOverlay';
```

화면 렌더링 순서를 다음과 같이 변경한다.

```tsx
return (
  <Screen onLayout={handleScreenLayout}>
    <VentingSceneBackground
      frame={backgroundFrame}
      progress={homeBackgroundProgress}
    />

    {isOnboardingCompleted ? <VentingSunlightOverlay /> : null}

    <Container
      style={
        isOnboardingCompleted
          ? undefined
          : {
              paddingTop: insets.top,
              paddingBottom: Math.max(insets.bottom, 24),
            }
      }>
      {/* 기존 내용 유지 */}
    </Container>
  </Screen>
);
```

온보딩 완료 후 보이는 실제 메인 숲 화면에만 햇빛을 표시한다. 온보딩 전 화면에도
필요하다면 조건문을 제거하고 `<VentingSunlightOverlay />`를 항상 렌더링한다.

## 구현 원리

각 `LinearGradient`는 가로 방향으로 다음 순서의 색을 가진다.

1. 완전 투명
2. 약한 금빛
3. 가장 밝은 금빛
4. 약한 금빛
5. 완전 투명

이 구조로 별도 블러 없이도 광선 양쪽 가장자리가 부드럽게 사라진다. 광선은
화면보다 위아래로 크게 렌더링하므로 회전할 때 직사각형의 끝부분이 화면 안에
보이지 않는다.

세 광선은 각각 다른 폭, 위치, 회전을 가지지만 애니메이션은 바깥쪽
`movingGroup`에 한 번만 적용한다. 따라서 세 개의 독립 애니메이션보다 설정과
정리가 단순하고 Reanimated UI 스레드에서 동작한다.

## 접근성 및 생명주기

- `useIsFocused()`가 `false`이면 반복 애니메이션을 취소한다.
- 시스템에서 동작 줄이기가 활성화되면 가운데 위치의 정적인 빛만 유지한다.
- `ReduceMotion.System`을 애니메이션 설정에도 전달한다.
- 컴포넌트가 정리될 때 `cancelAnimation()`을 호출한다.
- 접근성 트리에서 장식용 광선을 숨긴다.
- `pointerEvents="none"`으로 터치를 가로채지 않는다.

## 새 배경 이미지 적용 시 주의사항

제공된 새 배경 이미지와 현재 공용 긴 배경의 크기가 다르다.

| 구분 | 크기 |
| --- | --- |
| 새 `배경3.png` | `839×1718` |
| 현재 `img_background_forest_long.png` | `402×1470` |
| 현재 기본 배경 기준 | `402×894` |

현재 긴 배경은 메인 화면, 메인→채팅 전환 오버레이, 채팅 화면에서 함께 사용된다.
화면 전환 계산에도 높이 `1470`이 고정값으로 들어가 있다.

따라서 새 이미지를 기존 긴 배경 파일에 그대로 덮어쓰면 안 된다.
`resizeMode="contain"`으로 인해 큰 여백이 생기거나 화면 전환 좌표가 어긋날 수
있다.

### 권장 방법

새 그림을 기존 좌표계와 동일한 `402×1470` 긴 배경으로 다시 내보낸다.

- 상단의 메인 숲 구도를 유지한다.
- 하단은 채팅 화면 전환에 사용할 수 있도록 자연스럽게 연장한다.
- 기존 기준 너비와 높이를 유지한다.
- 새 에셋이 준비되면 `VentingBackgroundLong`만 교체한다.
- 기존 레이아웃 상수와 장면 전환 계산은 1차에서 변경하지 않는다.

긴 배경 에셋을 바로 준비할 수 없다면 기존 배경에 햇빛 애니메이션만 먼저 적용하고,
배경 교체를 후속 작업으로 분리하는 편이 안전하다.

## 1차 완료 기준

- [ ] 실행 후 1~2초 안에 햇빛의 움직임을 인지할 수 있다.
- [ ] 빛이 흰색이 아닌 따뜻한 금빛으로 보인다.
- [ ] 돌멩이, 버튼, 텍스트의 밝기와 가독성이 유지된다.
- [ ] 버튼과 화면 제스처가 정상 동작한다.
- [ ] 화면을 벗어나면 애니메이션이 정지한다.
- [ ] 동작 줄이기 사용 시 반복 움직임이 없다.
- [ ] 메인→채팅 전환 시 배경이 튀거나 위치가 바뀌지 않는다.
- [ ] 작은 Android 화면과 긴 iPhone 화면에서 광선 끝이 노출되지 않는다.
- [ ] iOS 실제 기기와 Android 실제 기기에서 각각 확인한다.

## 2차 디테일 후보

1차 구현과 QA가 완료된 뒤 아래 항목을 개별적으로 추가한다.

### 광선별 미세 움직임

- 각 광선에 8초, 11초, 14초처럼 서로 다른 주기를 사용한다.
- 밝아지는 타이밍을 서로 다르게 설정한다.
- 이동 거리는 광선별로 ±4~10px 이내로 제한한다.

### 넓은 주변광

- 화면 중앙 상단에 알파가 낮은 금빛 글로우를 추가한다.
- 글로우는 6~10초 주기로 약 10%만 밝아졌다 흐려지게 한다.
- 메인 UI의 대비가 낮아지면 글로우를 제거한다.

### 나뭇잎 그림자

- 알파가 낮은 반투명 그림자 에셋을 광선 반대 방향으로 움직인다.
- 주기는 12~16초로 광선보다 느리게 설정한다.
- Android에서 블렌드 모드 차이가 발생하지 않도록 에셋 자체에 알파를 적용한다.

### 장면 전환 연결

- 메인→채팅 전환이 시작될 때 햇빛을 150~250ms 동안 페이드아웃한다.
- 채팅 화면까지 햇빛을 유지할 필요가 있는지 제품 판단 후 결정한다.

### Lottie 사용 조건

디자이너가 빛의 형태와 타이밍을 직접 반복 수정해야 하고, 정적인 그라데이션
레이어로 원하는 질감을 만들 수 없을 때만 Lottie로 전환한다. 단순한 흔들림과
밝기 변화만 필요하다면 Reanimated 구현을 유지한다.

## 관련 코드

- `pebbling-expo/package.json`
- `src/screens/venting/VentingHomeScreen.tsx`
- `src/screens/venting/VentingSceneBackground.tsx`
- `src/screens/venting/ventingSceneTransition.ts`
- `src/styles/images.ts`

## 공식 문서

- [Expo SDK 56](https://docs.expo.dev/versions/v56.0.0/)
- [Expo LinearGradient](https://docs.expo.dev/versions/v56.0.0/sdk/linear-gradient/)
- [Reanimated withRepeat](https://docs.swmansion.com/react-native-reanimated/docs/animations/withRepeat/)
- [Reanimated 접근성](https://docs.swmansion.com/react-native-reanimated/docs/guides/accessibility/)
