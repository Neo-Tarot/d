# Audio & Haptic Feedback Implementation

## Overview

This document describes the immersive audio and haptic feedback system implemented for Neo-Tarot AI. The system enhances the mystical atmosphere with sound effects, background music, and tactile feedback.

## Features Implemented

### 1. Haptic Feedback System

- **Custom Hook**: `useAppHaptics` provides convenient access to Telegram haptic patterns
- **Integration Points**:
  - Card selection: Light haptic (selection changed)
  - Card flip: Heavy haptic (magic reveal)
  - Button clicks: Medium haptic (selection changed)
  - Reading reveal: Success notification haptic
  - Shuffle animation: Light ticks during shuffle

### 2. Audio System

- **Context-based Architecture**: `SoundManagerProvider` wraps the entire app
- **Background Music (BGM)**:
  - Looping ambient music
  - Auto-starts on first user interaction (bypasses browser autoplay policies)
  - Default volume: 30%
  - Respects mute state
- **Sound Effects (SFX)**:
  - `card_flip`: Triggered when card flips
  - `shuffle`: Triggered when deck is shuffled
  - `reveal_magic`: Triggered when "Reveal Destiny" is clicked
  - `button_click`: Triggered on navigation/button interactions
- **Audio Pool**: Uses 3 instances per SFX for overlapping playback

### 3. User Controls

- **Mute Toggle**: Located in Profile > Settings > Sound & Music
- **Persistent State**: User preference stored in localStorage (`neo-tarot-sound-muted`)
- **Visual Feedback**: Shows 🔊/🔇 icon and ON/OFF status

## File Structure

```
src/
├── shared/
│   └── lib/
│       └── audio/
│           ├── SoundManager.tsx      # Main audio context & provider
│           ├── useAppHaptics.ts      # Haptic feedback hook
│           └── index.ts              # Exports
├── features/
│   └── deck/
│       └── lib/
│           └── haptics.ts            # Low-level haptic functions (existing)
├── pages/
│   ├── HomePage/
│   │   └── HomePage.tsx              # Updated with audio/haptics
│   ├── ReadingPage/
│   │   └── ReadingPage.tsx           # Updated with audio/haptics
│   └── ProfilePage/
│       └── ProfilePage.tsx           # Updated with mute toggle
└── app/
    └── Root.tsx                      # SoundManagerProvider integrated

public/
└── sounds/
    ├── README.md                     # Audio file requirements
    ├── bgm_ambient.mp3
    ├── flip.mp3
    ├── shuffle.mp3
    ├── magic.mp3
    └── click.mp3
```

## Integration Points

### ReadingPage

- **Shuffle Deck**: `shuffle.mp3` + light haptic ticks
- **Flip Card**: `flip.mp3` + heavy haptic after animation
- **Reveal Destiny Button**: `magic.mp3` + success haptic
- **Reset Button**: `click.mp3` + selection haptic

### HomePage

- **Start Reading Button**: `click.mp3` + selection haptic

### ProfilePage

- **Restore Energy Button**: `click.mp3` + selection haptic
- **Sound Toggle Button**: Haptic only (no SFX when toggling mute)

### Card Component

- **Auto-triggers**: Plays `flip.mp3` when `isFlipped` prop changes
- **Timing**: Sound plays immediately, haptic after 600ms animation

### Deck Component

- **Auto-triggers**: Plays `shuffle.mp3` when shuffle starts
- **Haptic Pattern**: 4 light ticks at 0ms, 150ms, 300ms, 450ms

## Usage Example

```tsx
import { useAppHaptics, useSoundManager } from "@/shared/lib/audio";

function MyComponent() {
	const { playSFX, isMuted, toggleMute } = useSoundManager();
	const haptics = useAppHaptics();

	const handleClick = () => {
		playSFX("button_click");
		haptics.buttonClick();
		// ... your logic
	};

	return <button onClick={handleClick}>Click Me</button>;
}
```

## Audio File Requirements

See `/public/sounds/README.md` for detailed audio file specifications:

- Format: MP3 (best browser compatibility)
- Sample Rate: 44.1 kHz
- Bit Rate: 128-192 kbps
- File sizes: Keep SFX under 100KB each

## Browser Compatibility

- **Autoplay Policy**: BGM starts on first user interaction to comply with browser autoplay restrictions
- **Haptics**: Only available in Telegram mobile apps (iOS, Android)
- **Graceful Degradation**: App works without audio files or haptics

## Testing Checklist

- [ ] Add audio files to `/public/sounds/`
- [ ] Navigate to Profile > Settings
- [ ] Toggle "Sound & Music" to ON
- [ ] Test Reading Flow:
  - [ ] Tap deck (shuffle sound + haptics)
  - [ ] Flip 3 cards (flip sound + haptic per card)
  - [ ] Click "Reveal Destiny" (magic sound + success haptic)
- [ ] Test Navigation:
  - [ ] Click "Start Reading" (click sound + haptic)
  - [ ] Click "New Reading" (click sound + haptic)
- [ ] Test Mute Toggle:
  - [ ] Toggle to OFF (sounds stop, haptics continue)
  - [ ] Refresh page (setting persists)

## Performance Considerations

- **Audio Preloading**: All SFX preloaded on mount
- **Memory Management**: Audio instances cleaned up on unmount
- **Pool Pattern**: 3 instances per SFX prevents audio interruption
- **Lightweight**: No external audio libraries, uses native Web Audio API

## Future Enhancements (Not Implemented)

- Volume sliders for BGM and SFX separately
- Additional sound themes
- Vibration API fallback for non-Telegram environments
- Audio visualization during reading reveal
- Custom haptic patterns for different card arcanas

## Troubleshooting

**BGM doesn't start:**

- Check browser console for autoplay policy errors
- Ensure user has interacted with page (click/tap)
- Verify `bgm_ambient.mp3` exists in `/public/sounds/`

**SFX not playing:**

- Check browser console for 404 errors
- Verify all SFX files exist in `/public/sounds/`
- Check if mute is enabled in Profile settings

**Haptics not working:**

- Haptics only work in Telegram mobile apps
- Desktop/web preview will not trigger haptics (expected behavior)
- Check Telegram app settings for haptic feedback enabled
