# Real-Time Pose Recommendation System

An AI-powered fitness application that provides instant feedback on exercise form using real-time pose detection. Leverages MediaPipe for body keypoint detection, TensorFlow.js for client-side processing, and Supabase for data persistence.

## Features

### Core Functionality
- **Real-Time Pose Detection**: MediaPipe Pose model detects 33 body keypoints at 30+ FPS
- **Exercise Form Analysis**: Analyzes squat, push-up, lunge, and plank exercises with specific form rules
- **Multi-Modal Feedback**:
  - Visual skeleton overlay with color-coded confidence levels
  - Real-time text recommendations for form corrections
  - Confidence percentage display for each body part
  - Audio alerts and voice guidance for critical form issues
- **Session Tracking**: Records and analyzes pose data throughout workout sessions
- **Performance Analytics**: Dashboard with charts showing progress over time

### Supported Exercises
1. **Squat**: Monitors knee alignment, hip angle, back angle, and weight distribution
2. **Push-up**: Tracks elbow angle, body straightness, and shoulder engagement
3. **Lunge**: Analyzes front/back knee angles, torso position, and weight balance
4. **Plank**: Monitors body alignment and core engagement

## Technical Architecture

### Frontend (Next.js 16)
- **Pose Detection**: `lib/pose-detector.ts` - MediaPipe integration with canvas rendering
- **Form Analysis**: `lib/form-analyzer.ts` - Exercise-specific rules engine with angle calculations
- **Audio Feedback**: `lib/audio-feedback.ts` - Web Audio API for beeps and Web Speech API for voice
- **Components**:
  - `PoseDetector`: Main detection loop with real-time visualization
  - `FeedbackOverlay`: Real-time feedback messages on video feed
  - `ConfidenceIndicator`: Body part confidence breakdown
  - `SessionTracker`: Session history management
  - `ExerciseSelector`: Exercise selection interface

### Backend (Next.js API Routes)
- **Database Setup**: `app/api/setup-db/route.ts` - Initialize Supabase tables
- **Session Management**: `app/api/sessions/route.ts` - CRUD operations for exercise sessions
- **Supabase Client**: `lib/supabase-client.ts` - Database abstraction layer

### Database (Supabase PostgreSQL)
```sql
Tables:
- exercises: Exercise definitions with form rules
- exercise_sessions: Workout session records
- pose_frames: Individual frame data (sampled every 1s)
- form_recommendations: Form feedback history
- session_statistics: Aggregated user analytics
```

## Installation

### Prerequisites
- Node.js 18+
- pnpm package manager
- Supabase account (free tier available)
- Modern browser with webcam access

### Setup Steps

1. **Clone and Install**
```bash
git clone <repo-url>
cd pose-recommendation
pnpm install
```

2. **Environment Variables**
Create `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key
```

3. **Initialize Database**
```bash
# The database tables are created automatically via the setup API
# Visit http://localhost:3000/api/setup-db (POST request)
```

4. **Start Development Server**
```bash
pnpm dev
```

Access the app at `http://localhost:3000`

## Usage

### Running a Session
1. Select an exercise from the selector
2. Click "Start Detection" to enable camera
3. Position yourself in frame - the system detects your pose in real-time
4. Follow on-screen feedback to correct form issues
5. Click "Stop Detection" when finished

### Interpreting Feedback
- **Green skeleton points**: High confidence detection
- **Red/yellow messages**: Form corrections needed
- **Audio beeps**: 
  - High pitch: Success (good form)
  - Medium pitch: Warning (minor issue)
  - Low pitch: Critical (major issue)
- **Voice alerts**: Verbal guidance for critical form issues

### Analytics Dashboard
- View performance metrics across all sessions
- Compare exercise performance with charts
- Export session data as CSV
- Track confidence improvements over time

## Form Analysis Details

### Squat Analysis
- **Target angles**: Knees 60-110°, hips 70-100°, back lean 15-45°
- **Critical checks**: Knee alignment, back straightness, weight centering
- **Common issues**: Knees caving inward, excessive forward lean

### Push-up Analysis
- **Target angles**: Elbows 45-90°, shoulder angle 60-90°
- **Critical checks**: Body alignment, elbow tracking, shoulder engagement
- **Common issues**: Body sagging, elbows flaring out

### Lunge Analysis
- **Target angles**: Front knee 70-110°, back knee 30-90°, torso lean 10-30°
- **Critical checks**: Front knee alignment, back knee depth, upright torso
- **Common issues**: Front knee extending past ankle, inadequate back knee depth

### Plank Analysis
- **Target angles**: Body angle 0-10° (perfectly straight)
- **Critical checks**: Straight body, core engagement, neutral neck
- **Common issues**: Sagging hips, hiking hips, head position

## Performance Optimization

### Real-Time Processing
- **Web Workers**: Pose detection runs in background thread
- **Throttled Analysis**: Form analysis updates every 100ms (not every frame)
- **Frame Sampling**: Database saves frames every 1 second
- **GPU Acceleration**: TensorFlow.js uses WebGL when available

### Browser Compatibility
- Chrome/Edge 90+: Full support with GPU acceleration
- Firefox 88+: Supported (CPU fallback)
- Safari 14+: Supported with reduced performance
- Mobile: Limited support due to camera API constraints

## API Reference

### POST `/api/sessions`
Create or manage exercise sessions

**Create Session**:
```json
{
  "action": "create",
  "sessionData": {
    "exerciseId": "uuid",
    "userId": "uuid|null"
  }
}
```

**Save Frame**:
```json
{
  "action": "saveFrame",
  "frameData": {
    "sessionId": "uuid",
    "frameNumber": 1,
    "keypoints": [...],
    "jointAngles": {...},
    "formFeedback": [...],
    "formConfidence": 0.85,
    "bodyPartConfidence": {...}
  }
}
```

### GET `/api/sessions?action=exercises`
Fetch available exercises

## Advanced Configuration

### Adjusting Form Rules
Edit `lib/form-analyzer.ts` `EXERCISE_RULES` object:
```typescript
const EXERCISE_RULES = {
  'Custom Exercise': {
    target_angles: {
      joint_name: { min: 30, max: 90 }
    },
    critical_points: ['check_name']
  }
};
```

### Customizing Feedback Thresholds
Modify confidence thresholds in `lib/form-analyzer.ts`:
- Change `calculateAngleConfidence()` range calculation
- Adjust feedback severity in `analyzeFormForExercise()`

### Audio Customization
Adjust frequencies and timing in `lib/audio-feedback.ts`:
- `playAudioCue()`: Modify oscillator frequencies
- `speakFeedback()`: Change speech rate and pitch

## Troubleshooting

### Camera Not Working
- Check browser permissions: Settings → Privacy → Camera
- Ensure HTTPS or localhost (camera requires secure context)
- Try a different browser

### Poor Pose Detection
- Ensure adequate lighting (natural light preferred)
- Keep entire body in frame
- Maintain 2-3 feet distance from camera
- Avoid reflective backgrounds

### Slow Performance
- Close other browser tabs
- Reduce video quality if available
- Try a different browser with better GPU support
- Check browser's hardware acceleration is enabled

### No Database Connection
- Verify Supabase credentials in `.env.local`
- Check internet connection
- Ensure Supabase project is active
- Run setup API route manually: `curl -X POST http://localhost:3000/api/setup-db`

## Future Enhancements

- Multi-person pose detection for group classes
- Rep counting and workout templates
- Pose comparison with reference videos
- Integration with fitness wearables
- Mobile app with offline capability
- AI coach with personalized workout plans
- Social features (leaderboards, sharing)

## Performance Benchmarks

- **Detection Speed**: 30-60 FPS on modern laptops
- **Latency**: 50-100ms from motion to feedback
- **Accuracy**: 85-95% confidence on well-framed poses
- **Memory**: 150-200MB for pose detection model

## Security & Privacy

- All pose detection runs locally in the browser
- Video stream never stored or transmitted
- Optional cloud storage for session analytics
- User sessions can be deleted anytime
- No facial recognition or identification

## License

Personal

## Support

For issues, questions, or contributions, please open an issue on GitHub or contact support.

## Credits

Built with:
- [MediaPipe](https://mediapipe.dev/) - Pose detection
- [TensorFlow.js](https://www.tensorflow.org/js) - ML runtime
- [Next.js](https://nextjs.org/) - Framework
- [Supabase](https://supabase.com/) - Database
- [shadcn/ui](https://ui.shadcn.com/) - UI components
- [Recharts](https://recharts.org/) - Analytics charts
