# CampusTrack UI Components Guide

This guide documents all the new standardized UI components and how to use them.

## Design System Files

### Design Tokens (`src/config/designTokens.ts`)
Central configuration for all design values:
- **Spacing**: `xs` (4px), `sm` (8px), `md` (16px), `lg` (24px), `xl` (32px), `xxl` (48px)
- **Typography**: Predefined font sizes, weights, and line heights (h1-h4, body, label, caption)
- **Shadows**: `none`, `sm`, `md`, `lg`, `xl` elevation levels
- **Border Radius**: `xs` (4px), `sm` (8px), `md` (12px), `lg` (16px), `xl` (20px), `full` (circular)
- **Animations**: Timing constants (`fast`, `base`, `slow`, `slower`)
- **Colors**: Light and dark theme color palettes with semantic colors (success, warning, error, info)

**Usage:**
```tsx
import { spacing, typography, shadows, borderRadius } from '../config/designTokens';

// Example
<View style={{ padding: spacing.md, borderRadius: borderRadius.lg, ...shadows.md }}>
```

---

## UI Components

### 1. Button Component
Reusable button with multiple variants and states.

**Import:**
```tsx
import { Button } from '../components';
// or
import Button from '../components/Button';
```

**Props:**
```tsx
interface ButtonProps {
  variant?: 'primary' | 'secondary' | 'outline' | 'text' | 'danger'; // default: 'primary'
  size?: 'small' | 'medium' | 'large';          // default: 'medium'
  loading?: boolean;                             // Shows spinner
  disabled?: boolean;                            // Disabled state
  onPress: () => void;                           // Required callback
  children: string;                              // Button text
  icon?: keyof MaterialCommunityIcons;           // Optional icon
  iconPosition?: 'left' | 'right';               // default: 'left'
  fullWidth?: boolean;                           // Stretches to fill width
  testID?: string;                               // For testing
  accessibilityLabel?: string;                   // For accessibility
}
```

**Examples:**
```tsx
// Primary button
<Button onPress={() => console.log('Pressed')}>
  Click Me
</Button>

// Button with icon and loading state
<Button 
  variant="primary" 
  size="large"
  icon="send"
  loading={isLoading}
  onPress={handleSubmit}
>
  Submit Assignment
</Button>

// Outline button
<Button variant="outline" onPress={handleCancel}>
  Cancel
</Button>

// Danger button (red)
<Button variant="danger" onPress={handleDelete}>
  Delete Profile
</Button>

// Full width button
<Button fullWidth onPress={handleLogin}>
  Sign In
</Button>

// Text-only button
<Button variant="text" onPress={handleLearnMore}>
  Learn More
</Button>
```

**Features:**
- ✅ Haptic feedback on press
- ✅ Scale animation when pressed
- ✅ Loading spinner overlay
- ✅ Fully accessible with labels and states
- ✅ All 5 variants (primary, secondary, outline, text, danger)
- ✅ 3 sizes (small, medium, large)

---

### 2. TextInput Component
Standardized text input with validation, error display, and helper text.

**Import:**
```tsx
import { TextInput } from '../components';
// or
import TextInput from '../components/TextInput';
```

**Props:**
```tsx
interface TextInputProps {
  label?: string;                              // Field label
  placeholder?: string;                        // Placeholder text
  value: string;                               // Current value (required)
  onChangeText: (text: string) => void;       // Change callback (required)
  error?: string;                              // Error message
  helperText?: string;                         // Helper/hint text
  icon?: keyof MaterialCommunityIcons;         // Left icon
  rightIcon?: keyof MaterialCommunityIcons;    // Right icon
  onRightIconPress?: () => void;               // Right icon callback
  disabled?: boolean;                          // Disable input
  secureTextEntry?: boolean;                   // Password field
  size?: 'small' | 'medium' | 'large';         // default: 'medium'
  variant?: 'filled' | 'outlined';             // default: 'outlined'
  multiline?: boolean;                         // Multi-line text
  numberOfLines?: number;                      // Line count
  maxLength?: number;                          // Max characters
  showCharacterCount?: boolean;                // Show char count
  required?: boolean;                          // Show required indicator
  testID?: string;                             // For testing
}
```

**Examples:**
```tsx
// Basic input
const [email, setEmail] = useState('');
<TextInput
  label="Email"
  placeholder="Enter your email"
  value={email}
  onChangeText={setEmail}
/>

// Input with validation
const [password, setPassword] = useState('');
const [passwordError, setPasswordError] = useState('');

<TextInput
  label="Password"
  value={password}
  onChangeText={setPassword}
  secureTextEntry
  error={passwordError}
  helperText="At least 8 characters"
  required
/>

// Input with icon
<TextInput
  label="Search"
  icon="magnify"
  placeholder="Search assignments..."
  value={search}
  onChangeText={setSearch}
/>

// Multiline textarea
<TextInput
  label="Description"
  multiline
  numberOfLines={4}
  maxLength={500}
  showCharacterCount
  value={description}
  onChangeText={setDescription}
/>

// Filled variant
<TextInput
  label="Name"
  variant="filled"
  value={name}
  onChangeText={setName}
/>
```

**Features:**
- ✅ Error state with red color and icon
- ✅ Helper text below field
- ✅ Character count with warning at 80% capacity
- ✅ Password toggle eye icon
- ✅ Left and right icons
- ✅ Required field indicator (*)
- ✅ 2 variants (filled, outlined)
- ✅ 3 sizes (small, medium, large)
- ✅ Full accessibility support
- ✅ Focus animations

---

### 3. Card Component
Reusable card container with multiple variants.

**Import:**
```tsx
import { Card } from '../components';
// or
import Card from '../components/Card';
```

**Props:**
```tsx
interface CardProps {
  variant?: 'elevated' | 'filled' | 'outlined';  // default: 'elevated'
  onPress?: () => void;                          // Optional press callback
  children: React.ReactNode;                     // Card content
  style?: ViewStyle;                             // Custom styles
  testID?: string;                               // For testing
  disabled?: boolean;                            // Disable card
  padding?: 'xs' | 'sm' | 'md' | 'lg' | 'xl';   // default: 'md'
}
```

**Examples:**
```tsx
// Basic card
<Card>
  <Text>Card content here</Text>
</Card>

// Pressable card
<Card onPress={() => navigation.navigate('Details')}>
  <Text>Tap to see details</Text>
</Card>

// Card with custom padding
<Card padding="lg">
  <Text style={{ fontSize: 18, fontWeight: '600' }}>Large Card</Text>
</Card>

// Filled variant
<Card variant="filled">
  <Text>Filled background</Text>
</Card>

// Outlined variant
<Card variant="outlined">
  <Text>Outlined style</Text>
</Card>
```

**Features:**
- ✅ 3 variants (elevated, filled, outlined)
- ✅ Optional press handling
- ✅ Shadow/elevation management
- ✅ Customizable padding
- ✅ Disabled state support
- ✅ Full accessibility

---

### 4. Skeleton Component
Loading placeholder component with shimmer animation.

**Import:**
```tsx
import { 
  Skeleton, 
  CardSkeleton, 
  ListSkeleton, 
  DashboardSkeleton 
} from '../components';
```

**Props:**
```tsx
interface SkeletonProps {
  width?: number | string;                    // default: '100%'
  height?: number | string;                   // default: 20
  variant?: 'rectangular' | 'circular' | 'text';  // default: 'rectangular'
  style?: ViewStyle;                          // Custom styles
  testID?: string;                            // For testing
}
```

**Examples:**
```tsx
// Simple skeleton
<Skeleton width="100%" height={20} />

// Circular skeleton (avatar)
<Skeleton variant="circular" width={60} height={60} />

// Text skeletons
<Skeleton variant="text" width="60%" />
<Skeleton variant="text" width="100%" />

// Prebuilt card skeleton
<CardSkeleton count={3} />

// Prebuilt list skeleton
<ListSkeleton count={5} />

// Prebuilt dashboard skeleton
<DashboardSkeleton />

// Using in conditional render
{loading ? (
  <DashboardSkeleton />
) : (
  <Dashboard data={data} />
)}
```

**Features:**
- ✅ Shimmer animation (auto-loops)
- ✅ 3 variants (rectangular, circular, text)
- ✅ Prebuilt loaders for common patterns
- ✅ Responsive sizing
- ✅ Lightweight and performant

---

## How to Update Existing Screens

### Before (Old way):
```tsx
<TouchableOpacity
  style={{
    backgroundColor: '#1B72B5',
    padding: 20,
    borderRadius: 12,
    justifyContent: 'center',
    alignItems: 'center',
  }}
  onPress={handleSubmit}
>
  <Text style={{ color: '#fff', fontSize: 16, fontWeight: '600' }}>
    Submit
  </Text>
</TouchableOpacity>
```

### After (New way):
```tsx
import { Button } from '../components';

<Button onPress={handleSubmit}>
  Submit
</Button>
```

---

## Color Palette Integration

### Light Theme
```tsx
import { lightTheme } from '../config/designTokens';

// Colors available:
lightTheme.primary       // #1B72B5
lightTheme.success       // #10B981
lightTheme.warning       // #F59E0B
lightTheme.error         // #EF4444
lightTheme.background    // #FFFFFF
lightTheme.text          // #1F2937
lightTheme.border        // #E5E7EB
```

### Dark Theme
```tsx
import { darkTheme } from '../config/designTokens';

// Same properties, optimized for dark mode
darkTheme.background     // #000000
darkTheme.text           // #FFFFFF
```

---

## Testing Components

All components support `testID` for testing:

```tsx
// Button testing
<Button testID="login-button" onPress={handleLogin}>
  Login
</Button>

// TextInput testing
<TextInput 
  testID="email-input"
  value={email}
  onChangeText={setEmail}
/>

// Card testing
<Card testID="profile-card">
  <Text>Profile</Text>
</Card>

// Access in tests
fireEvent.press(getByTestId('login-button'));
expect(getByTestId('email-input')).toHaveValue('test@email.com');
```

---

## Accessibility

All components include full accessibility support:

- ✅ `accessibilityLabel` - Screen reader labels
- ✅ `accessibilityRole` - Semantic roles (button, text, etc.)
- ✅ `accessibilityState` - Current state (disabled, loading, etc.)
- ✅ `accessibilityHint` - Additional hints for screen readers

**Example:**
```tsx
<Button
  accessibilityLabel="Save profile changes"
  accessibilityHint="Double tap to save"
  onPress={handleSave}
>
  Save
</Button>
```

---

## Migration Guide

### Step 1: Import Components
Replace scattered component files with imports from the new components folder:

```tsx
// Old
import Button from './Button';
import Input from './Input';

// New
import { Button, TextInput } from '../components';
```

### Step 2: Update Props
Refer to the props documentation above and update component usage.

### Step 3: Test
Test each screen after updating:
- Tap buttons and verify they work
- Type in inputs and verify validation
- Check both light and dark modes
- Test on different screen sizes

### Step 4: Gradual Rollout
Don't update everything at once. Update screen by screen:
1. Dashboard
2. StudentSettings/Profile
3. EditProfile
4. Attendance
5. Assignment
6. Chat

---

## Performance Considerations

- **Skeletons**: Use skeletons instead of blank screens for better perceived performance
- **Animations**: All animations use `useNativeDriver: true` for 60fps performance
- **Heavy Lists**: Use `ListSkeleton` for loading states
- **Button Loading**: Use `loading` prop instead of disabling button

---

## Browser/Platform Support

- ✅ Android 5.0+
- ✅ iOS 11.0+
- ✅ Web (React Native Web)

---

## Common Patterns

### Form Submission
```tsx
const [isLoading, setIsLoading] = useState(false);
const [error, setError] = useState('');

<TextInput
  label="Email"
  value={email}
  onChangeText={setEmail}
  error={error}
/>

<Button
  loading={isLoading}
  onPress={async () => {
    setIsLoading(true);
    try {
      await submitForm();
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  }}
>
  Submit
</Button>
```

### Loading State
```tsx
{loading ? (
  <DashboardSkeleton />
) : (
  <Dashboard data={data} />
)}
```

### Validation
```tsx
const [email, setEmail] = useState('');
const [emailError, setEmailError] = useState('');

const validateEmail = (email: string) => {
  if (!email.includes('@')) {
    setEmailError('Invalid email address');
  } else {
    setEmailError('');
  }
};

<TextInput
  label="Email"
  value={email}
  onChangeText={(text) => {
    setEmail(text);
    validateEmail(text);
  }}
  error={emailError}
/>
```

---

## Support & Documentation

For more information:
- See `UI_UX_AUDIT_REPORT.md` for design standards
- Check component source files for detailed TypeScript types
- Review examples in `src/screens/` for real-world usage

---

