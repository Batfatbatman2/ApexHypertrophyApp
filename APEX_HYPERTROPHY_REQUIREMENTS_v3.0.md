APEX_HYPERTROPHY_REQUIREMENTS_v3.0.md
# Apex Hypertrophy:  Complete Requirements Document

**Document Overview**

| Field | Value |
|-------|-------|
| Version | 3.0 |
| Last Updated | December 15, 2025 |
| Status | Draft - Pending Review |
| Platform | React Native (iOS & Android) |
| Architecture | Offline-First, Local Storage Only |

---

## Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | Dec 14, 2024 | Initial draft |
| 2.0 | Dec 14, 2024 | Added NFRs, error handling, accessibility, backup strategy, acceptance criteria, state management, edge cases, success metrics |
| 2.1 | Dec 15, 2024 | Moved backup/export to Phase 2, added sync conflict resolution notes, added ML cold start optimization note, aligned muscle group enums |
| 3.0 | Dec 15, 2025 | **Major revision**:  Complete overhaul of ML system to Hypertrophy AI Coach architecture.  Added SFR system, personalized volume management, intelligent exercise selection, proactive deload scheduling, program generation, real-time session adaptation, enhanced user profiling, multi-model ML architecture |

---

## Table of Contents

1. Project Vision
2. Glossary & Definitions
3. Constraints & Assumptions
4. Out of Scope
5. Technical Stack
6. Non-Functional Requirements
7. Data Model
8. State Management
9. User Profile System
10. Core Features
11. Hypertrophy AI Coach System
12. Stimulus-to-Fatigue Ratio (SFR) Engine
13. Intelligent Volume Management
14. Exercise Selection & Swap Engine
15. Proactive Deload System
16. Intelligent Program Generation
17. Real-Time Session Adaptation
18. ML Architecture & Models
19. Error Handling & Recovery
20. UI/UX Specifications
21. Accessibility Requirements
22. Data Backup & Recovery
23. Implementation Phases
24. Testing Requirements
25. Success Metrics
26. Deployment Checklist
27. Appendices

---

## 1. Project Vision

### 1.1 Core Purpose

Apex Hypertrophy is a zero-friction strength training app that removes all mental overhead from workout logging and programming. The app functions as an **AI-powered hypertrophy coach** that learns your body's responses and optimizes every aspect of your training—exercise selection, volume, intensity, and recovery—like having a sport scientist in your pocket.

### 1.2 Key Differentiators

| Feature | Description |
|---------|-------------|
| **Hypertrophy AI Coach** | Personalized coaching powered by multi-model ML system trained on exercise science |
| **Stimulus-to-Fatigue Optimization** | Ranks and selects exercises based on YOUR response, not generic recommendations |
| **Adaptive Volume Management** | Automatically adjusts sets per muscle group based on your recovery and growth response |
| **Proactive Deload Scheduling** | Predicts fatigue before you plateau, not after |
| **Intelligent Exercise Swapping** | Automatically suggests alternatives when exercises stop working or cause pain |
| **Bad Week Detection** | Recognizes off days/weeks and adjusts expectations accordingly |
| **Atomic Auto-Save** | Data saves instantly on every input (zero data loss) |
| **Rolling Schedule** | Flexible workout queue (not calendar-locked) |
| **Offline-Only** | No internet required, ever |
| **Accessible by Default** | Full VoiceOver/TalkBack support |

### 1.3 Design Philosophy

- **AI as Coach, Not Controller**:  Suggestions are always explainable and overridable
- **Learn From Every Rep**: Every data point improves personalization
- **Respect User Preferences**: Favorite exercises are protected, dislikes are avoided
- **Numbers First**: Large, tappable inputs optimized for gym use
- **Dark Mode Default**: Slate grey backgrounds (OLED-optimized)
- **Apex Red for Action**: Primary actions only (Start, Complete)
- **Minimal Chrome**: No unnecessary UI elements during workout
- **Graceful Degradation**: App functions fully even when ML fails

### 1.4 Target Users

| Persona | Description | Key Needs | AI Coach Benefits |
|---------|-------------|-----------|-------------------|
| **Beginner Lifter** | 0-1 years experience | Pre-built programs, simple logging, guidance | Conservative progression, technique-focused suggestions |
| **Intermediate Lifter** | 1-3 years experience | Customization, progression tracking | Optimized volume, plateau-breaking swaps |
| **Advanced Lifter** | 3+ years experience | Detailed analytics, flexible programming | Fine-tuned SFR optimization, advanced periodization |

---

## 2. Glossary & Definitions

### 2.1 Fitness Terms

| Term | Definition |
|------|------------|
| **RPE (Rate of Perceived Exertion)** | A 1-10 scale measuring how hard a set felt.  RPE 10 = absolute failure, RPE 7 = could do 3 more reps |
| **Rep (Repetition)** | One complete movement of an exercise |
| **Set** | A group of consecutive repetitions |
| **Working Set** | A set performed at target intensity (not warm-up) |
| **Warm-up Set** | A lighter set to prepare muscles and joints |
| **Drop Set** | Immediately reducing weight after a set to continue |
| **Myo-reps** | A rest-pause technique with short rest between mini-sets |
| **Deload** | A planned reduction in training intensity/volume for recovery |
| **Plateau** | A period where progress stalls despite consistent training |
| **Volume** | Total work performed (weight × reps × sets) |
| **1RM (One Rep Max)** | The maximum weight you can lift for one repetition |
| **Compound Exercise** | Movement using multiple joints (e.g., squat, bench press) |
| **Isolation Exercise** | Movement using a single joint (e.g., bicep curl) |
| **Muscle Connection** | Subjective feeling of targeting the intended muscle |
| **SFR (Stimulus-to-Fatigue Ratio)** | Ratio of muscle growth stimulus to systemic/local fatigue caused |
| **MEV (Minimum Effective Volume)** | Minimum sets per week needed to stimulate growth |
| **MAV (Maximum Adaptive Volume)** | Optimal volume range ceiling for growth |
| **MRV (Maximum Recoverable Volume)** | Absolute maximum volume before overtraining |
| **MV (Maintenance Volume)** | Minimum volume to maintain current muscle mass |

### 2.2 Technical Terms

| Term | Definition |
|------|------------|
| **Atomic Save** | Data persisted immediately on each input, not batched |
| **Draft Session** | An in-progress workout that hasn't been completed |
| **Rolling Schedule** | Program days cycle in order, not tied to calendar days |
| **Cold Start** | Initial period where ML has insufficient data |
| **Source of Truth** | The authoritative data source (SQLite in our case) |
| **SFR Profile** | User-specific stimulus and fatigue scores for an exercise |
| **Volume Landmark** | Personalized volume thresholds (MEV, MAV, MRV) per muscle |
| **Fatigue Index** | Calculated score representing accumulated training stress |
| **Bad Week** | Period where external factors cause underperformance |

---

## 3. Constraints & Assumptions

### 3.1 Technical Constraints

| Constraint | Impact | Mitigation |
|------------|--------|------------|
| Offline-only architecture | No cloud sync, no social features | Local backup to device storage |
| On-device ML only | Model size limited, no cloud inference | Multi-model ensemble (~15MB total) with specialized lightweight models |
| SQLite single-writer | No concurrent write operations | Queue writes through Redux middleware |
| No internet requirement | Cannot fetch remote data | All exercise data and ML models bundled with app |
| Device processing power | Complex ML must run efficiently | Optimized inference, caching, background processing |

### 3.2 Business Constraints

| Constraint | Impact |
|------------|--------|
| Single developer/small team | Phased rollout, MVP focus |
| No backend infrastructure | No user accounts, no data recovery if device lost |
| App store approval required | Must meet Apple/Google guidelines |

### 3.3 Assumptions

| Assumption | Risk if Invalid |
|------------|-----------------|
| Users have devices with 2GB+ RAM | App may crash on low-end devices |
| Users understand basic gym terminology | Onboarding confusion |
| Users workout 3-6x per week | ML model trained on this frequency |
| Users primarily use one device | No sync = data loss on device switch |
| TensorFlow. js Lite works reliably on target devices | Must fallback to rule-based |
| Users will complete onboarding honestly | Poor initial recommendations |
| Users will provide RPE/feedback consistently | Reduced ML accuracy |

---

## 4. Out of Scope

### 4.1 Explicitly Excluded (MVP)

| Feature | Reason | Future Phase |
|---------|--------|--------------|
| Cloud sync | Complexity, cost, privacy concerns | Phase 8+ |
| Social features | Against core philosophy | Never |
| Nutrition tracking | Different domain, many competitors | Phase 9+ |
| Cardio tracking | Focus is hypertrophy | Phase 8+ |
| Apple Watch app | Development resources | Phase 7 |
| Light mode | OLED optimization priority | Phase 7 |
| Multi-language support | MVP English only | Phase 7 |
| Video demonstrations | Storage constraints | Phase 7 |
| Coach/trainer accounts | B2B complexity | Phase 8+ |
| Voice commands | Complexity | Phase 8+ |
| Barcode scanning (plates) | Hardware dependency | Phase 9+ |

### 4.2 Future Considerations (Document Now, Implement Later)

| Feature | Notes |
|---------|-------|
| **Sync Conflict Resolution** | Use last-write-wins with vector clocks for simple conflicts.  For workout sessions, prefer the version with more logged sets. Store `updated_at` timestamp on all mutable records.  |
| **Merge Strategy** | Completed workouts are immutable and always sync. Draft workouts only exist on one device. |
| **Multi-Device Support** | When cloud sync is implemented, use device_id field to track origin of records. |
| **Federated Learning** | Future option to improve models across users without sharing raw data |

---

## 5. Technical Stack

### 5.1 Core Technologies

| Layer | Technology | Version | Justification |
|-------|------------|---------|---------------|
| Framework | React Native | 0.73. 2 (locked) | Cross-platform, mature ecosystem |
| State Management | Redux Toolkit | 2.0.1 (locked) | Predictable state, devtools |
| Local Database | SQLite | via react-native-sqlite-storage 6.0.1 | Reliable, performant, offline |
| Navigation | React Navigation | 6.1.9 (locked) | Industry standard |
| UI Components | Custom-built | N/A | Full control, no bloat |
| ML Framework | TensorFlow.js Lite | 4.15.0 (locked) | On-device inference |
| Charts | Victory Native | 36.9.1 (locked) | Performant, customizable |
| Background Tasks | react-native-background-fetch | 4.2.1 | Weekly model updates |

### 5.2 Device Requirements

| Platform | Minimum | Recommended |
|----------|---------|-------------|
| iOS Version | 15.0+ | 16.0+ |
| Android API | 28+ (Android 9) | 31+ (Android 12) |
| RAM | 3GB | 4GB+ |
| Storage (Initial) | 200MB | 200MB |
| Storage (Max) | 600MB | 600MB |

### 5.3 Performance Targets

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| Cold App Launch | < 2s | < 3s |
| Warm App Launch | < 500ms | < 1s |
| Input Response | < 50ms | < 100ms |
| Database Write | < 30ms | < 50ms |
| Database Read (simple) | < 10ms | < 20ms |
| Database Read (complex) | < 100ms | < 200ms |
| ML Inference (weight prediction) | < 100ms | < 200ms |
| ML Inference (full analysis) | < 500ms | < 1000ms |
| SFR Calculation | < 50ms | < 100ms |
| Screen Transition | < 200ms | < 300ms |
| Memory Usage (active) | < 250MB | < 350MB |
| Battery Drain | < 3%/hour | < 5%/hour |
| Frame Rate | 60fps | 55fps |

---

## 6. Non-Functional Requirements

### 6.1 Security Requirements

| ID | Requirement | Priority |
|----|-------------|----------|
| SEC-001 | Sensitive data must be encrypted at rest using SQLCipher or encrypted storage | High |
| SEC-002 | App must not transmit any data over network | High |
| SEC-003 | Biometric lock option for app access (FaceID/TouchID/Fingerprint) | Medium |
| SEC-004 | No analytics without explicit user consent | High |
| SEC-005 | Backup files must be encrypted with user-provided password | High |
| SEC-006 | Clear data option must be irreversible with confirmation | Medium |
| SEC-007 | ML models cannot be extracted or reverse-engineered easily | Medium |

### 6.2 Accessibility Requirements

| ID | Requirement | Priority | WCAG Level |
|----|-------------|----------|------------|
| A11Y-001 | All interactive elements must have accessible labels | High | A |
| A11Y-002 | Minimum touch target size of 44×44 points | High | AA |
| A11Y-003 | Color contrast ratio minimum 4.5:1 for text | High | AA |
| A11Y-004 | Color contrast ratio minimum 3:1 for UI components | High | AA |
| A11Y-005 | All functionality available via VoiceOver/TalkBack | High | A |
| A11Y-006 | No information conveyed by color alone | High | A |
| A11Y-007 | Support for Dynamic Type / Font Scaling | Medium | AA |
| A11Y-008 | Reduce Motion setting respected | Medium | AA |
| A11Y-009 | Screen reader announces state changes | High | A |
| A11Y-010 | Logical focus order in all screens | High | A |
| A11Y-011 | AI suggestions announced clearly with context | High | A |

### 6.3 Reliability Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| REL-001 | Crash-free session rate | ≥ 99.9% |
| REL-002 | ANR (App Not Responding) rate | < 0.1% |
| REL-003 | Data loss incidents | 0 |
| REL-004 | Successful workout completion rate | ≥ 99% |
| REL-005 | Draft recovery success rate | 100% |
| REL-006 | ML inference success rate | ≥ 99.5% |
| REL-007 | Graceful ML fallback rate | 100% |

### 6.4 Scalability Requirements

| ID | Requirement | Target |
|----|-------------|--------|
| SCALE-001 | Maximum exercises in library | 500+ |
| SCALE-002 | Maximum custom exercises per user | 100 |
| SCALE-003 | Maximum programs per user | 20 |
| SCALE-004 | Maximum workout history | 5+ years (~2000 sessions) |
| SCALE-005 | Maximum sets per workout | 100 |
| SCALE-006 | Database size limit | 600MB with warning at 500MB |
| SCALE-007 | ML model total size | ≤ 15MB |
| SCALE-008 | User exercise metrics records | Unlimited (auto-cleanup of stale data) |

---

## 7. Data Model

### 7.1 Entity Relationship Diagram

```
users (1) ──→ (1) user_profiles
users (1) ──→ (many) injury_records
users (1) ──→ (many) user_exercise_metrics
users (1) ──→ (many) user_volume_landmarks
users (1) ──→ (many) user_fatigue_log
users (1) ──→ (many) weekly_checkins
users (1) ──→ (many) programs
users (1) ──→ (many) workout_sessions
users (1) ──→ (many) deload_history
users (1) ──→ (many) exercise_swap_history
users (1) ──→ (many) ai_recommendations_log

programs (1) ──→ (many) program_days
program_days (1) ──→ (many) workout_templates
program_days (1) ──→ (many) workout_sessions

workout_sessions (1) ──→ (many) logged_sets
workout_sessions (1) ──→ (many) exercise_feedback
workout_sessions (1) ──→ (many) session_adaptations

exercises (1) ──→ (many) logged_sets
exercises (1) ──→ (many) user_exercise_metrics
exercises (1) ──→ (many) exercise_feedback
```

### 7.2 Core Tables

#### User & Profile Tables

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `users` | Base user record | id, name, created_at |
| `user_profiles` | Extended profile with training data | user_id, biological_sex, birth_date, height_cm, body_weight_kg, training_age_months, experience_level, primary_goal, available_days_per_week, session_duration_minutes, training_environment |
| `user_equipment` | Equipment availability | user_id, equipment_type, is_available |
| `user_preferences` | App and training preferences | user_id, weight_unit, rest_timer_enabled, biometric_lock_enabled, favorite_exercises (JSON), disliked_exercises (JSON) |
| `injury_records` | Injury and limitation history | user_id, body_part, injury_type, severity, affected_movements (JSON), date_occurred, date_resolved, is_active |
| `mobility_limitations` | Joint mobility issues | user_id, joint, limitation_type, severity, affected_exercises (JSON) |

#### Program & Workout Tables

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `programs` | Training programs | id, user_id, name, is_active, split_type, schedule_type, current_day_index, created_by ('user', 'ai', 'template'), ai_rationale |
| `program_days` | Days within a program | id, program_id, day_index, day_name, is_rest_day, target_muscles (JSON) |
| `exercises` | Exercise library | id, name, primary_muscle_group, secondary_muscle_groups (JSON), equipment, movement_pattern, is_compound, is_custom, base_stimulus_score, base_fatigue_score, base_joint_stress, strength_curve, primary_joint, secondary_joints (JSON) |
| `workout_templates` | Planned exercises per day | id, program_day_id, exercise_id, order_index, target_sets, target_reps_min, target_reps_max, ai_selected, selection_reason |
| `workout_sessions` | Completed/in-progress workouts | id, user_id, program_day_id, started_at, completed_at, is_draft, is_deload, total_volume, total_sets, duration_minutes, performance_rating, ai_adjustments_applied (JSON), draft_expires_at |
| `logged_sets` | Individual set data | id, workout_session_id, exercise_id, set_number, set_type, weight, reps, rpe, is_completed, is_pr, suggested_weight, user_overrode_suggestion, rest_seconds |
| `exercise_feedback` | RPE and quality tracking | id, workout_session_id, exercise_id, rpe, muscle_connection, joint_pain, pain_severity, pain_location, notes |

#### AI & ML Tables

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `user_exercise_metrics` | Personalized exercise data | user_id, exercise_id, user_stimulus_modifier, user_fatigue_modifier, user_sfr_ratio, progression_rate, plateau_count, weeks_since_progression, total_sessions, avg_muscle_connection, avg_rpe, optimal_rep_range_min, optimal_rep_range_max, pain_incidents, last_pain_date, confidence_score |
| `user_volume_landmarks` | Personalized volume thresholds | user_id, muscle_group, mv, mev, mav, mrv, current_weekly_volume, optimal_volume, volume_trend, best_growth_volume, data_points, confidence_score |
| `user_fatigue_log` | Fatigue tracking over time | user_id, recorded_at, overall_fatigue_score, rpe_creep, performance_decline, recovery_issues, motivation_level, sleep_quality, stress_level, days_since_deload, cumulative_volume_load |
| `weekly_checkins` | User-reported recovery data | user_id, week_start_date, body_weight, sleep_quality_avg, stress_level_avg, energy_level_avg, soreness_level, motivation_level, missed_sleep_nights, high_stress_days, notes |
| `deload_history` | Deload tracking and effectiveness | user_id, started_at, completed_at, protocol_type, intensity_modifier, volume_modifier, trigger_reason, fatigue_score_at_trigger, performance_before, performance_after, effectiveness_rating |
| `exercise_swap_history` | Exercise change tracking | user_id, swapped_at, old_exercise_id, new_exercise_id, swap_reason, old_exercise_sessions, old_exercise_sfr, new_exercise_sfr, outcome_rating |
| `ai_recommendations_log` | AI decision tracking | user_id, created_at, recommendation_type, recommendation_data (JSON), user_action, user_response_at, outcome_measured_at, outcome_positive, outcome_data (JSON) |
| `session_adaptations` | Real-time session changes | workout_session_id, adaptation_type, exercise_id, message, action_data (JSON), user_response, created_at |

#### System Tables

| Table | Purpose | Key Fields |
|-------|---------|------------|
| `db_migrations` | Schema version tracking | version, name, applied_at, checksum |
| `backups` | Backup history | id, filename, size_bytes, created_at, is_auto |
| `ml_model_versions` | Installed model tracking | model_name, version, installed_at, is_active |

### 7.3 Data Validation Rules

| Field | Validation | Error Message |
|-------|------------|---------------|
| `weight` | > 0 AND ≤ 2000 (lbs) / ≤ 907 (kg) | "Weight must be between 0 and 2000 lbs" |
| `reps` | ≥ 0 AND ≤ 100 | "Reps must be between 0 and 100" |
| `rpe` | ≥ 1 AND ≤ 10, increments of 0.5 | "RPE must be between 1 and 10" |
| `muscle_connection` | ≥ 1 AND ≤ 5 | "Rating must be between 1 and 5" |
| `pain_severity` | ≥ 1 AND ≤ 10 | "Severity must be between 1 and 10" |
| `rest_seconds` | ≥ 0 AND ≤ 600 | "Rest must be between 0 and 10 minutes" |
| `target_sets` | ≥ 1 AND ≤ 20 | "Sets must be between 1 and 20" |
| `draft_expires_at` | started_at + 12 hours | N/A (calculated) |
| `training_age_months` | ≥ 0 AND ≤ 600 | "Training age must be 0-50 years" |
| `available_days_per_week` | ≥ 2 AND ≤ 7 | "Training days must be 2-7" |
| `session_duration_minutes` | ≥ 30 AND ≤ 180 | "Session duration must be 30-180 minutes" |
| `sfr_ratio` | ≥ 0 AND ≤ 10 | N/A (calculated) |
| `confidence_score` | ≥ 0 AND ≤ 1 | N/A (calculated) |

### 7.4 Enums

#### Muscle Groups
```
CHEST, BACK, LATS, TRAPS, REAR_DELTS, SIDE_DELTS, FRONT_DELTS, 
BICEPS, TRICEPS, FOREARMS, QUADS, HAMSTRINGS, GLUTES, CALVES, ABS, OBLIQUES
```

#### Equipment Types
```
BARBELL, DUMBBELL, CABLE, MACHINE, BODYWEIGHT, KETTLEBELL, BANDS, 
EZ_BAR, TRAP_BAR, SMITH_MACHINE, OTHER
```

#### Movement Patterns
```
HORIZONTAL_PUSH, HORIZONTAL_PULL, VERTICAL_PUSH, VERTICAL_PULL,
HIP_HINGE, SQUAT, LUNGE, ISOLATION_CURL, ISOLATION_EXTENSION,
ISOLATION_RAISE, ISOLATION_FLY, CARRY, ROTATION
```

#### Training Splits
```
FULL_BODY, UPPER_LOWER, PUSH_PULL_LEGS, PUSH_PULL, 
ARNOLD_SPLIT, BRO_SPLIT, CUSTOM
```

#### Experience Levels
```
BEGINNER, INTERMEDIATE, ADVANCED, ELITE
```

#### Swap Reasons
```
PLATEAU, JOINT_PAIN, LOW_SFR, POOR_CONNECTION, USER_CHOICE, EQUIPMENT_UNAVAILABLE
```

#### Deload Protocol Types
```
INTENSITY_REDUCTION, VOLUME_REDUCTION, ACTIVE_RECOVERY, TARGETED_REST, STANDARD
```

#### AI Recommendation Types
```
WEIGHT_SUGGESTION, VOLUME_ADJUSTMENT, EXERCISE_SWAP, DELOAD_NEEDED,
PROGRAM_MODIFICATION, REST_DAY_SUGGESTED, FORM_CHECK_SUGGESTED
```

### 7.5 Migration Strategy

- All migrations tracked in `db_migrations` table with checksums
- Migrations must be reversible with stored rollback SQL
- Schema changes tested with existing user data before release
- Automatic migration on app launch with transaction safety
- ML model updates handled separately from schema migrations

---

## 8. State Management

### 8.1 Redux Store Structure

| Slice | Purpose | Persistence |
|-------|---------|-------------|
| `user` | Current user profile and preferences | SQLite (source of truth) |
| `programs` | Programs and program days | SQLite |
| `exercises` | Exercise library (read cache) | SQLite |
| `activeWorkout` | Current workout session state | SQLite + optimistic updates |
| `history` | Workout history and pagination | SQLite |
| `coach` | AI coach state, recommendations, suggestions | Ephemeral + SQLite for logs |
| `sfr` | SFR profiles cache | SQLite |
| `volume` | Volume landmarks and current tracking | SQLite |
| `fatigue` | Fatigue state and predictions | SQLite |
| `ui` | Theme, modals, toasts, keyboard state | AsyncStorage |

### 8.2 Source of Truth

| Data Type | Source of Truth | Redux Role |
|-----------|-----------------|------------|
| Workout sessions | SQLite | Cache for active session |
| Logged sets | SQLite | Optimistic updates, async persist |
| Exercises | SQLite | Read-only cache |
| Programs | SQLite | Cache with write-through |
| User exercise metrics | SQLite | Cache, updated after sessions |
| Volume landmarks | SQLite | Cache, recalculated weekly |
| Fatigue scores | SQLite | Cache, updated after sessions |
| AI suggestions | Calculated | Ephemeral cache |
| AI recommendation logs | SQLite | Write-through |
| UI state | Redux (persisted) | Primary store |

### 8.3 Data Flow

1. User interacts with UI component
2. Component dispatches action with optimistic update
3. Redux middleware queues database write (debounced 100ms)
4. SQLite write completes (< 30ms target)
5. Success action updates auto-save indicator
6. On failure:  retry with exponential backoff, show error toast if all retries fail
7. AI coach listens for relevant actions and triggers analysis
8. AI suggestions dispatched to coach slice
9. UI components subscribe to coach suggestions

### 8.4 AI Coach Data Flow

```
User Action (e.g., complete set)
        │
        ▼
Redux Middleware
        │
        ├──→ SQLite Write (logged_set)
        │
        └──→ AI Coach Service
                │
                ├──→ Feature Engineering
                │         │
                │         ▼
                ├──→ ML Model Inference
                │         │
                │         ▼
                ├──→ Post-Processing & Safety Bounds
                │         │
                │         ▼
                └──→ Dispatch Suggestion Action
                          │
                          ▼
                    UI Update (suggestion displayed)
```

---

## 9. User Profile System

### 9.1 Profile Data Structure

```typescript
interface UserProfile {
  // === IDENTITY ===
  id: string;
  name: string;
  created_at: Date;
  
  // === PHYSICAL ATTRIBUTES ===
  biological_sex: 'male' | 'female';
  birth_date: Date;
  height_cm: number;
  body_weight_kg:  number;
  body_fat_percentage?:  number;
  
  // === TRAINING BACKGROUND ===
  training_age_months: number;
  experience_level: 'beginner' | 'intermediate' | 'advanced' | 'elite';
  primary_goal: 'hypertrophy' | 'strength' | 'recomp' | 'general_fitness';
  secondary_goal?: 'hypertrophy' | 'strength' | 'recomp' | 'general_fitness';
  
  // === SCHEDULE ===
  available_days_per_week: number;
  preferred_session_duration_minutes: number;
  preferred_training_days: DayOfWeek[];
  
  // === EQUIPMENT ===
  training_environment: 'full_gym' | 'home_gym' | 'minimal' | 'bodyweight_only';
  available_equipment: EquipmentType[];
  
  // === INJURIES & LIMITATIONS ===
  injury_history: InjuryRecord[];
  mobility_limitations: MobilityLimitation[];
  exercises_to_avoid: string[];
  
  // === PREFERENCES ===
  favorite_exercises: string[];
  disliked_exercises: string[];
  preferred_rep_range: { min: number; max: number };
  
  // === ML-LEARNED PATTERNS ===
  recovery_capacity: RecoveryProfile;
  volume_tolerance: VolumeToleranceProfile;
}
```

### 9.2 Onboarding Flow

| Step | Screen | Required | Data Collected |
|------|--------|----------|----------------|
| 1 | Welcome | Yes | Name |
| 2 | Physical Stats | Yes | Sex, birth date, height, weight |
| 3 | Training Background | Yes | Training age, experience level, primary goal |
| 4 | Schedule | Yes | Available days, session duration, preferred days |
| 5 | Equipment | Yes | Training environment, available equipment |
| 6 | Injuries | No | Current injuries, mobility limitations |
| 7 | Preferences | No | Favorite exercises, exercises to avoid |
| 8 | Program Selection | Yes | Choose AI-generated or template program |

### 9.3 Profile Update Triggers

| Trigger | Data Updated | Frequency |
|---------|--------------|-----------|
| Weekly check-in | Body weight, sleep, stress, energy | Weekly |
| Post-workout | Recovery patterns, volume tolerance | After each session |
| Injury reported | Injury records, exercises to avoid | As needed |
| Manual edit | Any profile field | As needed |
| ML recalculation | Recovery capacity, volume tolerance | Weekly |

### 9.4 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-UP-001 | Onboarding completes in < 5 minutes |
| AC-UP-002 | All required fields validated before progression |
| AC-UP-003 | Profile data persists across app restarts |
| AC-UP-004 | Skipping optional steps doesn't block app use |
| AC-UP-005 | Profile editable from settings at any time |
| AC-UP-006 | Training age auto-increments monthly |
| AC-UP-007 | Body weight history tracked for trends |
| AC-UP-008 | Equipment changes trigger program review suggestion |

---

## 10. Core Features

### 10.1 Feature:  Workout Logging (Active Session)

#### User Story

As a lifter, I want to log my sets during a workout with AI-powered weight suggestions so that I can train optimally and track my progress.

#### User Flow

1. User taps "START WORKOUT" on Home Screen
2. App creates draft workout_session record with 12-hour expiration
3. AI Coach analyzes user state and applies any pre-workout adjustments
4. Full-screen modal opens showing first exercise with AI weight suggestion
5. User sees suggestion with confidence indicator and explanation
6. User enters weight/reps for each set (auto-saves within 50ms)
7. User marks set complete → rest timer starts, AI analyzes performance
8. After last set of exercise → RPE feedback modal appears
9. AI updates exercise metrics and prepares next suggestion
10. User completes all exercises and taps "FINISH WORKOUT"
11. AI calculates session summary, updates all metrics, triggers weekly analysis if needed
12. Draft converts to completed session, program advances to next day

#### Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-WL-001 | Workout session created as draft with 12-hour expiration |
| AC-WL-002 | Weight input pre-filled with AI suggestion (if available) |
| AC-WL-003 | AI suggestion shows confidence level (high/medium/low) |
| AC-WL-004 | AI suggestion shows brief explanation (expandable for details) |
| AC-WL-005 | User can override suggestion with one tap |
| AC-WL-006 | Override recorded for ML improvement |
| AC-WL-007 | Each input change persists to database within 50ms |
| AC-WL-008 | Auto-save indicator updates within 100ms of save completion |
| AC-WL-009 | Set completion triggers rest timer if enabled |
| AC-WL-010 | Final set completion triggers RPE modal |
| AC-WL-011 | All logged data survives app force quit |
| AC-WL-012 | Draft recovery notification appears on app launch if draft exists |
| AC-WL-013 | "Finish Workout" calculates total volume for completed sets only |
| AC-WL-014 | Completing workout triggers AI post-session analysis |
| AC-WL-015 | AI session adaptations displayed if triggered during workout |

#### Set States

| State | Visual | Behavior |
|-------|--------|----------|
| Pending | Grey, dimmed | Not interactive until reached |
| Active | Apex Red border, full opacity | Weight/reps inputs enabled, AI suggestion shown |
| Completed | Green checkmark, slightly dimmed | Can tap to edit |
| Skipped | Grey with skip icon | Excluded from calculations |

#### Set Types

| Type | Indicator | Purpose |
|------|-----------|---------|
| Working | None | Standard working sets |
| Warm-up | Yellow left border + flame icon | Preparation sets (excluded from progression) |
| Drop Set | Purple left border + arrow icon | Immediately follows working set |
| Myo-reps | Blue left border + pause icon | Rest-pause technique |
| Failure | Red left border | Set taken to failure |
| Back-off | Orange left border | Reduced weight set after heavy work |

### 10.2 Feature: RPE & Feedback Collection

#### User Story

As a lifter, I want to rate how each exercise felt so that the AI can personalize my training. 

#### User Flow

1. User completes final set of an exercise
2. Half-sheet modal slides up from bottom
3. Shows:  RPE slider (1-10), Muscle Connection slider (1-5), Joint Pain toggle
4. If Joint Pain = ON → expands to show Severity slider (1-10) and Location picker
5. User adjusts values or taps "Done" to use defaults
6. Data saves to exercise_feedback table
7. AI immediately updates user_exercise_metrics
8. Modal dismisses, next exercise becomes active

#### Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-RPE-001 | Modal appears automatically after final set completion |
| AC-RPE-002 | RPE slider range is 1-10 with 0.5 increments |
| AC-RPE-003 | Muscle connection slider range is 1-5 |
| AC-RPE-004 | Joint pain toggle reveals severity slider when ON |
| AC-RPE-005 | Default values (RPE=7, Muscle Connection=3, Pain=OFF) save if modal dismissed |
| AC-RPE-006 | Feedback saved to exercise_feedback table with session and exercise IDs |
| AC-RPE-007 | Modal is accessible via VoiceOver/TalkBack |
| AC-RPE-008 | Haptic feedback on slider value changes |
| AC-RPE-009 | Pain report triggers AI pain pattern analysis |
| AC-RPE-010 | Low muscle connection (1-2) for 3+ sessions triggers swap suggestion |

### 10.3 Feature: Rolling Schedule

#### User Story

As a lifter, I want my program to cycle through days in order so that I'm not locked to specific calendar days.

#### Behavior

| Action | Result |
|--------|--------|
| Complete workout | `current_day_index` advances to next day (wraps at end) |
| Insert rest day | Rest recorded in history, scheduled workout remains next |
| Skip workout | No automatic advancement (user must complete or insert rest) |
| AI suggests rest | Notification shown, user decides |

#### Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-RS-001 | Program advances after workout completion |
| AC-RS-002 | Index wraps to 0 after last day |
| AC-RS-003 | Insert rest day records rest without changing next workout |
| AC-RS-004 | Home screen shows correct next workout based on current_day_index |
| AC-RS-005 | Schedule preview shows next 7 days |
| AC-RS-006 | AI can suggest rest day insertion based on fatigue |

### 10.4 Feature: Warm-up Calculator

#### Warm-up Protocol

| Set | Percentage | Reps |
|-----|------------|------|
| 1 | 40% | 10 |
| 2 | 55% | 6 |
| 3 | 70% | 4 |
| 4 | 85% | 2 |

#### Behavior

- Weights round to nearest 5 lbs / 2.5 kg
- Minimum weight is bar weight (45 lbs / 20 kg) or minimum dumbbell
- Working weight pre-fills from AI suggestion or last session
- Warm-up sets insert before working sets with `set_type = 'warmup'`
- Number of warm-up sets scales with working weight (heavier = more warm-ups)

#### Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-WU-001 | Warm-up percentages follow protocol table |
| AC-WU-002 | Weights round to nearest practical increment |
| AC-WU-003 | Working weight pre-fills from AI suggestion |
| AC-WU-004 | Warm-ups insert before working sets |
| AC-WU-005 | Warm-up sets have set_type = 'warmup' |
| AC-WU-006 | Warm-up sets excluded from progression calculations |

### 10.5 Feature: Weekly Check-in

#### User Story

As a lifter, I want to report my recovery status weekly so that the AI can adjust my training accordingly.

#### User Flow

1. App prompts for check-in at start of first workout of the week
2. Quick survey (< 1 minute):
   - Current body weight (optional)
   - Average sleep quality this week (1-5)
   - Average stress level this week (1-5)
   - Average energy level this week (1-5)
   - Overall soreness level (1-5)
   - Any missed sleep nights?  (0-7)
   - Any high stress days? (0-7)
3. User submits or skips
4. AI incorporates data into fatigue calculations

#### Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-WC-001 | Check-in prompts once per calendar week |
| AC-WC-002 | Check-in can be skipped without blocking workout |
| AC-WC-003 | All ratings use consistent 1-5 scale |
| AC-WC-004 | Body weight tracked for trend analysis |
| AC-WC-005 | High stress/poor sleep flags trigger conservative AI suggestions |
| AC-WC-006 | Check-in data visible in analytics |

---

## 11. Hypertrophy AI Coach System

### 11.1 Overview

The Hypertrophy AI Coach is a multi-model system that acts as a personal trainer, making intelligent decisions about every aspect of training based on exercise science principles and the user's individual response patterns.

### 11.2 Core Capabilities

| Capability | Description | Models Used |
|------------|-------------|-------------|
| **Weight Suggestions** | Predict optimal weight for each set | Weight Prediction Model |
| **Volume Management** | Track and optimize sets per muscle group | Volume Optimization Model |
| **Exercise Selection** | Rank and select exercises by personalized SFR | Exercise Ranking Model |
| **Exercise Swapping** | Detect when to swap exercises and suggest alternatives | Plateau Detection + Exercise Ranking |
| **Deload Scheduling** | Predict fatigue and schedule deloads proactively | Fatigue Estimation + Deload Timing |
| **Program Generation** | Create fully personalized training programs | All models + Program Generator |
| **Session Adaptation** | Real-time adjustments during workouts | Fatigue Estimation + Weight Prediction |
| **Bad Week Detection** | Recognize and adjust for off periods | Fatigue Estimation |

### 11.3 Decision Hierarchy

```
1. SAFETY FIRST
   └─ Pain reported → Suggest swap or modification
   └─ Excessive fatigue → Suggest deload
   └─ Form concerns → Suggest weight reduction

2. USER PREFERENCES
   └─ Favorite exercises → Protected from auto-swap
   └─ Disliked exercises → Never suggested
   └─ Equipment availability → Filter all suggestions

3. OPTIMIZATION
   └─ SFR ranking → Best exercises for user
   └─ Volume landmarks → Optimal sets per muscle
   └─ Progression rate → Appropriate weight increases

4. PERSONALIZATION
   └─ Response patterns → Individual adjustments
   └─ Recovery capacity → Frequency/volume scaling
   └─ Training age → Progression speed
```

### 11.4 Suggestion Confidence Levels

| Level | Confidence Score | Visual | Behavior |
|-------|------------------|--------|----------|
| High | ≥ 80% | Green indicator | Shown prominently |
| Medium | 60-79% | Yellow indicator | Shown with explanation |
| Low | 40-59% | Grey indicator | Shown as "tentative" |
| Insufficient Data | < 40% | No indicator | Falls back to rule-based |

### 11.5 Fallback Hierarchy

```
1. ML Model Prediction (confidence ≥ 60%)
        │
        ▼ (if unavailable or low confidence)
2. Rule-Based Calculation
        │
        ▼ (if insufficient data)
3. Last Session Value
        │
        ▼ (if no history)
4. Conservative Default
```

### 11.6 Explainability Requirements

Every AI suggestion must include: 

| Element | Description | Example |
|---------|-------------|---------|
| **Quick Explanation** | One-line summary | "Small increase based on strong performance" |
| **Confidence Level** | Visual indicator | Green dot (87% confidence) |
| **Data Source** | Where suggestion came from | "AI Model" / "Smart Rules" / "Last Session" |
| **Detailed Breakdown** | Expandable factors | Last session stats, trend data, fatigue level |
| **Override Option** | Easy way to change | "+5" / "-5" buttons, manual entry |

### 11.7 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-AI-001 | All AI suggestions include explanation |
| AC-AI-002 | Confidence level displayed for all predictions |
| AC-AI-003 | User can override any AI suggestion |
| AC-AI-004 | Overrides recorded for model improvement |
| AC-AI-005 | AI failures fall back gracefully (no error shown to user) |
| AC-AI-006 | AI processing doesn't block UI (async) |
| AC-AI-007 | AI respects user preferences (favorites, dislikes) |
| AC-AI-008 | AI suggestions within safety bounds (±15% of last weight) |
| AC-AI-009 | AI adapts to bad weeks (conservative suggestions) |
| AC-AI-010 | AI recommendations logged for analysis |

---

## 12. Stimulus-to-Fatigue Ratio (SFR) Engine

### 12.1 Overview

The SFR Engine calculates personalized stimulus-to-fatigue ratios for each exercise based on the user's response, enabling intelligent exercise selection and swapping. 

### 12.2 SFR Components

| Component | Description | Scale | Data Sources |
|-----------|-------------|-------|--------------|
| **Base Stimulus Score** | Research-based muscle activation potential | 1-10 | Exercise database |
| **Base Fatigue Score** | Research-based systemic + local fatigue | 1-10 | Exercise database |
| **User Stimulus Modifier** | Adjustment based on muscle connection | -2 to +2 | RPE feedback |
| **User Fatigue Modifier** | Adjustment based on RPE efficiency | -2 to +2 | Session data |
| **Pain Penalty** | Reduction for pain-causing exercises | 0 to -3 | Pain reports |
| **Progression Bonus** | Increase for exercises with good progress | 0 to +1 | History analysis |

### 12.3 SFR Calculation Formula

```
User Stimulus Score = Base Stimulus + User Stimulus Modifier + Progression Bonus
User Fatigue Score = Base Fatigue + User Fatigue Modifier
Pain Penalty = f(pain_frequency, pain_severity, recency)

User SFR = (User Stimulus Score - Pain Penalty) / User Fatigue Score
```

### 12.4 SFR Confidence

| Data Points | Confidence | Treatment |
|-------------|------------|-----------|
| 0-2 sessions | Very Low (< 30%) | Use base SFR only |
| 3-5 sessions | Low (30-50%) | Blend base and user SFR |
| 6-10 sessions | Medium (50-70%) | Weight toward user SFR |
| 11-20 sessions | High (70-85%) | Primarily user SFR |
| 21+ sessions | Very High (> 85%) | Full personalization |

### 12.5 SFR Update Triggers

| Trigger | Action |
|---------|--------|
| Set completed | Update running averages |
| RPE feedback submitted | Recalculate stimulus modifier |
| Pain reported | Apply pain penalty |
| Session completed | Full SFR recalculation |
| 4 weeks elapsed | Decay old data influence |

### 12.6 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-SFR-001 | SFR calculated for all exercises with ≥ 1 session |
| AC-SFR-002 | Base SFR from research data for all library exercises |
| AC-SFR-003 | User SFR updates after each session |
| AC-SFR-004 | Pain penalty applied within 24 hours of report |
| AC-SFR-005 | SFR confidence reflects data quality |
| AC-SFR-006 | SFR visible in exercise details (advanced view) |
| AC-SFR-007 | Exercise rankings sorted by SFR for selection |
| AC-SFR-008 | SFR data persists across app restarts |

---

## 13. Intelligent Volume Management

### 13.1 Overview

The Volume Management system tracks and optimizes the number of sets per muscle group per week, using personalized volume landmarks based on research and individual response. 

### 13.2 Volume Landmarks

| Landmark | Definition | Default Range (sets/week) |
|----------|------------|---------------------------|
| **MV (Maintenance Volume)** | Minimum to not lose gains | 4-8 |
| **MEV (Minimum Effective Volume)** | Minimum for growth | 6-12 |
| **MAV (Maximum Adaptive Volume)** | Optimal range ceiling | 12-20 |
| **MRV (Maximum Recoverable Volume)** | Absolute max before overtraining | 16-25 |

### 13.3 Default Landmarks by Muscle Group

| Muscle Group | MV | MEV | MAV | MRV |
|--------------|-----|-----|-----|-----|
| Chest | 6 | 10 | 18 | 22 |
| Back | 6 | 10 | 20 | 25 |
| Side Delts | 4 | 8 | 16 | 22 |
| Front Delts | 0 | 0 | 8 | 12 |
| Rear Delts | 0 | 6 | 14 | 18 |
| Biceps | 4 | 8 | 16 | 20 |
| Triceps | 4 | 6 | 14 | 18 |
| Quads | 6 | 8 | 16 | 20 |
| Hamstrings | 4 | 6 | 14 | 18 |
| Glutes | 0 | 4 | 12 | 16 |
| Calves | 4 | 8 | 16 | 20 |
| Abs | 0 | 6 | 16 | 20 |

### 13.4 Personalization Modifiers

| Factor | Impact | Range |
|--------|--------|-------|
| Training Age | Beginners need less | 0.7x - 1.0x |
| Biological Age | Older = less volume | 0.85x - 1.0x |
| Recovery Capacity | Poor recoverers need less | 0.8x - 1.0x |
| Sex | Females often tolerate more | 1.0x - 1.1x |
| Stress Level | High stress = less volume | 0.85x - 1.0x |
| Sleep Quality | Poor sleep = less volume | 0.9x - 1.0x |

### 13.5 Volume Status Classification

| Status | Condition | Action |
|--------|-----------|--------|
| **Below MEV** | Current < MEV | Suggest adding sets |
| **In Optimal Range** | MEV ≤ Current ≤ MAV | Monitor, suggest increases if responding |
| **Approaching MRV** | MAV < Current ≤ MRV | Monitor closely, warn about fatigue |
| **Exceeds MRV** | Current > MRV | Strongly suggest reducing sets |

### 13.6 Auto-Adjustment Rules

| Condition | Adjustment | Limit |
|-----------|------------|-------|
| Below MEV + Good recovery | Add 2 sets/week to lowest muscle | Max 4 sets/adjustment |
| Responding well + Room to grow | Add 1-2 sets to responding muscle | Max MAV |
| Exceeds MRV | Remove sets from lowest SFR exercise | Return to MAV |
| High fatigue + Approaching MRV | Suggest deload instead of reduction | - |

### 13.7 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-VOL-001 | Volume tracked per muscle group per week |
| AC-VOL-002 | Personalized landmarks calculated for each user |
| AC-VOL-003 | Volume status displayed in analytics |
| AC-VOL-004 | Suggestions generated when outside optimal range |
| AC-VOL-005 | Auto-adjustments require user confirmation |
| AC-VOL-006 | Volume changes logged for analysis |
| AC-VOL-007 | Compound exercises count toward all target muscles |
| AC-VOL-008 | Weekly volume recalculated every Monday |

---

## 14. Exercise Selection & Swap Engine

### 14.1 Overview

The Exercise Swap Engine detects when exercises should be changed and suggests optimal alternatives based on SFR rankings and user preferences.

### 14.2 Swap Triggers

| Trigger | Condition | Urgency |
|---------|-----------|---------|
| **Joint Pain** | Pain reported ≥ 2 times in 4 sessions | Immediate |
| **Plateau** | < 2. 5% progress over 4+ weeks AND RPE increasing | Suggested |
| **Low SFR** | User SFR < 0.8 AND confidence > 70% AND better option exists | Optional |
| **Poor Connection** | Avg muscle connection < 2 for 4+ sessions | Suggested |
| **User Request** | Manual swap initiated | Immediate |

### 14.3 Bad Week Detection

Before triggering plateau-based swaps, check for "bad week" indicators:

| Indicator | Threshold | Action if Detected |
|-----------|-----------|-------------------|
| RPE elevated across exercises | +1 point vs average | Suppress swap, note bad week |
| Multiple exercises underperforming | > 50% of exercises | Suppress swap, suggest rest |
| Poor sleep/high stress reported | Check-in data | Suppress swap, conservative mode |

### 14.4 Alternative Selection Criteria

Alternatives ranked by: 

1. **SFR Score** - Higher is better
2. **Movement Pattern Match** - Same pattern preferred
3. **Equipment Availability** - Must be available
4. **User History** - Tried before vs new
5. **Pain History** - No historical pain
6. **User Preferences** - Not in disliked list

### 14.5 Protected Exercises

Favorite exercises are protected from automatic swapping UNLESS: 
- Causing joint pain (user prompted with options)
- User explicitly requests swap

### 14.6 Swap Flow

```
Trigger Detected
      │
      ▼
Bad Week Check ──→ [Bad Week] ──→ Suppress, Log, Suggest Rest
      │
      ▼ [Normal]
Protected Check ──→ [Protected] ──→ Notify Only (unless pain)
      │
      ▼ [Not Protected]
Generate Alternatives (top 3 by SFR)
      │
      ▼
Present to User with Explanation
      │
      ▼
User Decision:  Swap / Keep / Deload Exercise / More Options
      │
      ▼
Log Decision, Update Program if Swapped
```

### 14.7 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-SWP-001 | Pain triggers swap suggestion within 24 hours |
| AC-SWP-002 | Plateau requires minimum 4 weeks of data |
| AC-SWP-003 | Bad weeks don't trigger swap suggestions |
| AC-SWP-004 | Alternatives filtered by equipment availability |
| AC-SWP-005 | Alternatives exclude disliked exercises |
| AC-SWP-006 | Favorite exercises protected unless pain |
| AC-SWP-007 | Swap decisions logged for ML improvement |
| AC-SWP-008 | Swap outcomes tracked (was new exercise better?) |
| AC-SWP-009 | User can always decline swap suggestion |
| AC-SWP-010 | Top 3 alternatives shown with explanations |

---

## 15. Proactive Deload System

### 15.1 Overview

The Deload System predicts when users need recovery weeks BEFORE plateaus occur, using fatigue indicators and historical patterns. 

### 15.2 Fatigue Indicators

| Indicator | Description | Weight |
|-----------|-------------|--------|
| **RPE Creep** | RPE increasing while weights stay same | 25% |
| **Performance Decline** | Weights/reps dropping | 25% |
| **Recovery Issues** | More rest needed between sets/sessions | 20% |
| **Motivation Drop** | Skipping workouts, cutting sessions short | 15% |
| **Pain Frequency** | More joint pain reports | 15% |

### 15.3 Fatigue Score Calculation

```
Fatigue Score = (
  RPE_Creep × 0.25 +
  Performance_Decline × 0.25 +
  Recovery_Issues × 0.20 +
  Motivation_Drop × 0.15 +
  Pain_Frequency × 0.15
)

Range: 0.0 (fully recovered) to 1.0 (extremely fatigued)
```

### 15.4 Deload Urgency Classification

| Urgency | Condition | Recommendation |
|---------|-----------|----------------|
| **Immediate** | Fatigue > 0.8 OR weeks since deload > base + 2 | "Deload this week" |
| **Soon** | Fatigue > 0.6 OR ≤ 1 week until recommended | "Consider deloading soon" |
| **Scheduled** | ≤ 2 weeks until recommended | "Deload in X weeks" |
| **Not Needed** | > 2 weeks until recommended | "Keep pushing" |

### 15.5 Deload Protocol Types

| Protocol | When Used | Intensity | Volume |
|----------|-----------|-----------|--------|
| **Intensity Reduction** | RPE creep, performance decline | 60% | 100% |
| **Volume Reduction** | Recovery issues | 85% | 50% |
| **Active Recovery** | Motivation drop | 50% | 50% |
| **Targeted Rest** | Pain frequency | 70% (unaffected) | 70% (unaffected) |
| **Standard** | General fatigue | 70% | 70% |

### 15.6 Deload Duration

- Default: 1 week
- Extended (if still fatigued): 2 weeks
- Targeted rest: Until pain subsides or 2 weeks max

### 15.7 Deload Effectiveness Tracking

After deload completion, measure: 
- Performance vs pre-deload (should improve)
- RPE vs pre-deload (should decrease)
- Pain frequency (should decrease)
- User-reported energy/motivation

### 15.8 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-DL-001 | Fatigue score calculated after each session |
| AC-DL-002 | Deload prediction updated daily |
| AC-DL-003 | Immediate urgency triggers prominent notification |
| AC-DL-004 | User can start deload with one tap |
| AC-DL-005 | Deload protocol auto-adjusts workout weights |
| AC-DL-006 | Deload week clearly marked in UI |
| AC-DL-007 | Post-deload effectiveness measured |
| AC-DL-008 | User can skip/postpone deload suggestion |
| AC-DL-009 | Deload history tracked for pattern learning |
| AC-DL-010 | Optimal deload frequency personalized over time |

---

## 16. Intelligent Program Generation

### 16.1 Overview

The Program Generator creates fully personalized training programs based on user profile, goals, equipment, schedule, and learned response patterns.

### 16.2 Program Generation Inputs

| Input | Source | Impact |
|-------|--------|--------|
| Available days | User profile | Split type selection |
| Session duration | User profile | Exercise count per session |
| Experience level | User profile | Volume, progression rate |
| Primary goal | User profile | Exercise selection, rep ranges |
| Equipment | User profile | Exercise filtering |
| Injuries | User profile | Exercise exclusion |
| Favorite exercises | User profile | Priority inclusion |
| Disliked exercises | User profile | Exclusion |
| SFR rankings | ML system | Exercise selection |
| Volume landmarks | ML system | Sets per muscle |
| Recovery capacity | ML system | Frequency, volume |

### 16.3 Split Selection Logic

| Days Available | Experience | Recommended Split |
|----------------|------------|-------------------|
| 2 | Any | Full Body |
| 3 | Beginner | Full Body |
| 3 | Intermediate+ | Push/Pull/Legs or Full Body |
| 4 | Beginner | Upper/Lower |
| 4 | Intermediate+ | Upper/Lower or Push/Pull/Legs |
| 5 | Any | Upper/Lower + 1 or Push/Pull/Legs + 2 |
| 6 | Any | Push/Pull/Legs 2x |
| 7 | Any | Push/Pull/Legs 2x + 1 |

### 16.4 Exercise Selection Process

```
For each muscle group:
  1. Get target volume from landmarks
  2. Get ranked exercises by user SFR
  3. Include favorite exercises first (if any)
  4. Fill remaining with highest SFR exercises
  5. Ensure variety (different movement patterns)
  6. Respect equipment constraints
  7. Avoid injury-affected movements
  8. Distribute across training days
```

### 16.5 Volume Distribution

| Split Type | Muscle Frequency | Sets Per Session |
|------------|------------------|------------------|
| Full Body | 3x/week | Lower (4-6 per muscle) |
| Upper/Lower | 2x/week | Medium (6-10 per muscle) |
| Push/Pull/Legs | 2x/week | Medium-High (8-12 per muscle) |
| Bro Split | 1x/week | High (12-16 per muscle) |

### 16.6 Progression Scheme Assignment

| Experience | Goal | Progression Type |
|------------|------|------------------|
| Beginner | Any | Linear (add weight each session) |
| Intermediate | Hypertrophy | Double progression (reps then weight) |
| Intermediate | Strength | Weekly linear |
| Advanced | Any | Periodized (volume waves) |

### 16.7 Program Rationale

Generated programs include explanation: 
- Why this split was chosen
- Why each exercise was selected
- Volume justification per muscle
- Progression scheme explanation
- Customizations applied (favorites, avoided exercises, injury accommodations)

````markdown name=APEX_HYPERTROPHY_REQUIREMENTS_v3.0.md (continued)
### 16.8 Program Modification

Programs can be modified by AI or user: 

| Modification Type | Trigger | Process |
|-------------------|---------|---------|
| **Exercise Swap** | Plateau, pain, low SFR | AI suggests, user confirms |
| **Volume Adjustment** | Outside optimal range | AI suggests, user confirms |
| **Add Exercise** | Muscle below MEV | AI suggests exercise + placement |
| **Remove Exercise** | Muscle above MRV | AI suggests which to remove |
| **Reorder Exercises** | User preference | Manual drag-and-drop |
| **Change Sets/Reps** | User preference | Manual edit |

### 16.9 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-PG-001 | Program generated within 5 seconds |
| AC-PG-002 | Generated program respects all user constraints |
| AC-PG-003 | Favorite exercises included when possible |
| AC-PG-004 | Disliked exercises never included |
| AC-PG-005 | Injured movements avoided |
| AC-PG-006 | Volume within personalized landmarks |
| AC-PG-007 | Rationale provided for all major decisions |
| AC-PG-008 | User can regenerate with different parameters |
| AC-PG-009 | User can edit any aspect of generated program |
| AC-PG-010 | Program modifications logged for analysis |
| AC-PG-011 | Template programs available as starting points |

---

## 17. Real-Time Session Adaptation

### 17.1 Overview

The Real-Time Adaptation system monitors workout performance and makes intelligent adjustments during the session. 

### 17.2 Adaptation Triggers

| Trigger | Detection Method | Response |
|---------|------------------|----------|
| **Bad Day Detected** | ≥ 3 sets underperforming by > 10% | Suggest reducing remaining weights by 10% |
| **Exceptional Day** | ≥ 3 sets overperforming | Suggest increasing remaining weights by 5% |
| **Performance Drop (Single Exercise)** | > 20% drop between sets | Suggest stopping exercise or reducing weight |
| **Pain Reported** | User reports pain mid-workout | Offer immediate swap |
| **Extended Rest Needed** | User extends rest timer multiple times | Note fatigue, adjust expectations |
| **Crushing It** | All sets at low RPE, top of rep range | Suggest adding volume |

### 17.3 Bad Day Score Calculation

```
Bad Day Score = Underperforming Sets / Total Completed Sets

Underperforming = Actual Volume < Expected Volume × 0.9

If Bad Day Score > 0.5 → Trigger adaptation
```

### 17.4 Adaptation Types

| Type | Visual | User Action Required |
|------|--------|---------------------|
| **Suggestion** | Toast notification | Dismiss or accept |
| **Warning** | Yellow banner | Acknowledge |
| **Alert** | Modal dialog | Must respond |
| **Automatic** | Silent adjustment | None (logged) |

### 17.5 Adaptation Presentation

```typescript
interface SessionAdaptation {
  type: 'reduce_intensity' | 'increase_intensity' | 'performance_warning' | 
        'pain_detected' | 'volume_suggestion' | 'fatigue_warning';
  urgency: 'suggestion' | 'warning' | 'alert';
  message: string;
  exercise_id?:  string;
  action:  {
    type: string;
    data:  any;
  };
  dismissible:  boolean;
}
```

### 17.6 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-RT-001 | Bad day detection after minimum 3 completed sets |
| AC-RT-002 | Adaptations don't interrupt active set entry |
| AC-RT-003 | Adaptations appear between sets or exercises |
| AC-RT-004 | Pain-triggered swaps offered immediately |
| AC-RT-005 | All adaptations dismissible (except pain alerts) |
| AC-RT-006 | Adaptations logged for analysis |
| AC-RT-007 | User can disable real-time suggestions in settings |
| AC-RT-008 | Adaptations respect user's decision (no repeated prompts) |

---

## 18. ML Architecture & Models

### 18.1 Multi-Model System Overview

| Model | Purpose | Architecture | Size | Inference Time |
|-------|---------|--------------|------|----------------|
| **Weight Prediction** | Suggest optimal weight | Gradient Boosting | 2MB | < 100ms |
| **Plateau Detection** | Identify stalled exercises | Random Forest | 1MB | < 50ms |
| **Fatigue Estimation** | Predict recovery needs | LSTM | 3MB | < 150ms |
| **Volume Optimization** | Optimal sets per muscle | Neural Network | 2MB | < 100ms |
| **Exercise Ranking** | Personalized SFR | Neural Network | 2MB | < 100ms |
| **Deload Timing** | When to schedule deload | Gradient Boosting | 1MB | < 50ms |
| **Total** | - | - | **~11MB** | - |

### 18.2 Weight Prediction Model

#### Input Features (31 features)

| Category | Features | Count |
|----------|----------|-------|
| Exercise Context | category, is_compound, movement_pattern | 3 |
| Historical Performance | last 4 weights, reps, RPE | 12 |
| Trends | weight, reps, RPE, volume trends (4 weeks) | 4 |
| User Context | training_age, days_since_last, fatigue_score, sleep, stress, is_deload | 6 |
| Exercise-Specific | sfr_ratio, progression_rate, plateau_risk, optimal_rep_center | 4 |
| Session Context | sets_completed_today, exercises_completed_today | 2 |

#### Output

```typescript
interface WeightPrediction {
  suggested_weight: number;
  confidence: number; // 0-1
  explanation_factors: {
    name: string;
    impact: 'increase' | 'decrease' | 'neutral';
    weight:  number; // Feature importance
  }[];
}
```

#### Safety Bounds

| Bound | Value |
|-------|-------|
| Maximum increase | +10% from last session |
| Maximum decrease | -15% from last session |
| Minimum confidence to show | 40% |
| Rounding | Nearest 2. 5 lbs / 1 kg |

### 18.3 Plateau Detection Model

#### Input Features (18 features)

| Category | Features | Count |
|----------|----------|-------|
| Performance History | last 8 sessions weight, reps, RPE | 24 → compressed to 8 |
| Trends | weight_change_4w, reps_change_4w, rpe_change_4w | 3 |
| Variability | weight_std, reps_std, rpe_std | 3 |
| Context | total_sessions, weeks_at_current_weight | 2 |
| User Factors | training_age, experience_level | 2 |

#### Output

```typescript
interface PlateauPrediction {
  is_plateaued: boolean;
  probability: number; // 0-1
  plateau_type: 'weight' | 'reps' | 'both' | 'none';
  weeks_stalled: number;
  recommended_action: 'continue' | 'deload' | 'swap' | 'technique_focus' | 'volume_adjust';
}
```

### 18.4 Fatigue Estimation Model

#### Input Features (Time Series - 4 weeks)

| Category | Features | Count |
|----------|----------|-------|
| Weekly Volume | total_sets, total_volume_load | 8 |
| Weekly Intensity | avg_weight_percent, avg_rpe | 8 |
| Weekly Recovery | sleep_quality, stress_level, soreness | 12 |
| Performance | performance_vs_expected | 4 |
| User Factors | training_age, recovery_capacity, weeks_since_deload | 3 |

#### Output

```typescript
interface FatigueEstimation {
  current_fatigue: number; // 0-1
  fatigue_trend: 'increasing' | 'stable' | 'decreasing';
  projected_fatigue_1w: number;
  projected_fatigue_2w: number;
  contributing_factors: {
    factor: string;
    contribution: number;
  }[];
  deload_recommendation: {
    urgency: 'immediate' | 'soon' | 'scheduled' | 'not_needed';
    days_until_recommended: number;
  };
}
```

### 18.5 Volume Optimization Model

#### Input Features (Per Muscle Group)

| Category | Features | Count |
|----------|----------|-------|
| Current Volume | sets_per_week, volume_load | 2 |
| Response Data | progression_rate, avg_muscle_connection, avg_soreness | 3 |
| User Factors | training_age, recovery_capacity, stress_level | 3 |
| Historical | best_growth_volume, volume_at_plateau | 2 |

#### Output

```typescript
interface VolumeOptimization {
  muscle_group: MuscleGroup;
  current_volume: number;
  optimal_volume: number;
  mev:  number;
  mav: number;
  mrv: number;
  recommendation: 'increase' | 'maintain' | 'decrease';
  sets_to_adjust: number;
  confidence: number;
}
```

### 18.6 Model Training Strategy

#### Pre-Training (Bundled with App)

| Aspect | Value |
|--------|-------|
| Data Source | Synthetic data based on exercise science research |
| Sample Size | 50,000 synthetic training histories |
| Research Basis | Prilepin's Table, NSCA Guidelines, Renaissance Periodization, Stronger By Science |
| Validation | Cross-validated against published studies |

#### On-Device Personalization

| Aspect | Value |
|--------|-------|
| Method | Transfer learning / Fine-tuning |
| Update Frequency | Weekly (background) |
| Minimum Data | 20 sessions for personalization |
| Privacy | All data stays on device |

#### Continuous Improvement

| Mechanism | Description |
|-----------|-------------|
| Override Tracking | Record when user changes AI suggestion |
| Outcome Tracking | Measure if suggestion led to good performance |
| Confidence Calibration | Adjust confidence based on prediction accuracy |
| Feature Importance | Track which factors matter most for each user |

### 18.7 Model Fallback Hierarchy

```
For each prediction type: 

1. Personalized ML Model (if 20+ sessions, confidence ≥ 60%)
   │
   ▼ (if unavailable)
2. Base ML Model (pre-trained, confidence ≥ 50%)
   │
   ▼ (if unavailable)
3. Rule-Based Calculation
   │
   ▼ (if insufficient data)
4. Conservative Default / Last Value
```

### 18.8 ML Performance Targets

| Metric | Target | Measurement |
|--------|--------|-------------|
| Weight Prediction MAE | < 5 lbs | Comparison to actual weight used |
| Weight Prediction within 5% | ≥ 80% | Accuracy metric |
| Weight Prediction within 10% | ≥ 95% | Accuracy metric |
| Plateau Detection Recall | ≥ 85% | Catch most plateaus |
| Plateau Detection Precision | ≥ 80% | Minimize false alarms |
| Fatigue Estimation Correlation | ≥ 0.75 | Correlation with actual performance |
| User Override Rate | < 30% | Measure suggestion quality |
| Suggestion Acceptance Rate | ≥ 70% | User satisfaction |

### 18.9 Acceptance Criteria

| ID | Criteria |
|----|----------|
| AC-ML-001 | All models load within 2 seconds on cold start |
| AC-ML-002 | Individual inference < 200ms |
| AC-ML-003 | Total model size < 15MB |
| AC-ML-004 | Models work fully offline |
| AC-ML-005 | Graceful fallback on any model failure |
| AC-ML-006 | No user-facing errors from ML system |
| AC-ML-007 | Personalization improves predictions over time |
| AC-ML-008 | User can reset personalization data |
| AC-ML-009 | Model updates don't interrupt active workouts |
| AC-ML-010 | Prediction explanations available for all suggestions |

---

## 19. Error Handling & Recovery

### 19.1 Error Classification

| Category | Severity | User Impact | Example |
|----------|----------|-------------|---------|
| Critical | P0 | Data loss possible | SQLite write failure during set logging |
| High | P1 | Feature unusable | ML model fails to load |
| Medium | P2 | Degraded experience | SFR calculation fails |
| Low | P3 | Minor inconvenience | Animation stutters |

### 19.2 Error Handling Strategies

| Error Type | Strategy | User Communication |
|------------|----------|-------------------|
| Database write failure | Retry 3x with exponential backoff, cache locally | "Saving..." → "Save failed, retrying..." |
| Database corruption | Attempt repair, offer restore from backup | "Data issue detected.  Attempting repair..." |
| ML model load failure | Fall back to rule-based (silent) | None (graceful degradation) |
| ML inference failure | Fall back to rule-based (silent) | None |
| SFR calculation failure | Use base SFR values | None (graceful degradation) |
| Volume calculation failure | Use default landmarks | None (graceful degradation) |
| Draft recovery failure | Offer to start fresh | "Couldn't recover previous workout.  Start new?" |
| Storage full | Warn user, prevent new workouts | "Device storage full. Please free up space." |
| Validation error | Inline error message, prevent submission | Field-specific error message |
| Network error (backup export) | Retry, save locally | "Export failed. Saved locally." |

### 19.3 Retry Policy

| Attempt | Delay | Action on Final Failure |
|---------|-------|------------------------|
| 1 | 100ms | Retry |
| 2 | 500ms | Retry |
| 3 | 1500ms | Cache locally, show error toast, queue for later |

### 19.4 Draft Recovery

| Scenario | Behavior |
|----------|----------|
| App reopened within 12 hours | Show dismissible notification with Resume button |
| App reopened after 12 hours | Auto-complete draft with logged sets, show info message |
| Draft recovery fails | Offer to start fresh workout |
| Multiple drafts exist | Show most recent, auto-complete older drafts |

### 19.5 AI Coach Error Handling

| Error | Handling | User Experience |
|-------|----------|-----------------|
| Weight prediction fails | Use last session weight | "Based on your last session" |
| SFR calculation fails | Use base SFR | Rankings still work, less personalized |
| Fatigue estimation fails | Disable proactive deload suggestions | Manual deload still available |
| Volume optimization fails | Use default landmarks | Generic volume recommendations |
| Exercise ranking fails | Alphabetical ordering | User can still select exercises |

### 19.6 Data Integrity Checks

| Check | Frequency | Action on Failure |
|-------|-----------|-------------------|
| Database checksum | On app launch | Attempt repair or restore |
| Foreign key integrity | On app launch | Log warning, attempt fix |
| ML model checksums | On model load | Redownload or use backup |
| User metrics consistency | Weekly | Recalculate from raw data |

---

## 20. UI/UX Specifications

### 20.1 Color Palette

| Category | Color | Hex | Usage |
|----------|-------|-----|-------|
| Background Primary | Slate Grey | #1C1C1E | Main app background |
| Background Secondary | Elevated Grey | #2C2C2E | Cards, surfaces |
| Background Tertiary | Input Grey | #3A3A3C | Input fields |
| Brand Primary | Apex Red | #FF3B30 | Primary CTAs only |
| Success | Green | #30D158 | Completed sets, positive feedback, high confidence |
| Warning | Yellow | #FFD60A | Warm-ups, alerts, medium confidence |
| Error | Red | #FF453A | Errors, destructive actions, pain indicators |
| Info | Blue | #0A84FF | Myo-reps, informational, AI suggestions |
| Purple | Purple | #BF5AF2 | Drop sets, deload indicator |
| Orange | Orange | #FF9500 | Back-off sets, caution |
| Text Primary | White | #FFFFFF | Primary text |
| Text Secondary | Grey | #98989D | Secondary text |
| Text Tertiary | Dark Grey | #636366 | Disabled, hints |
| AI Accent | Cyan | #64D2FF | AI coach indicators |

### 20.2 Typography

| Style | Size | Weight | Usage |
|-------|------|--------|-------|
| Hero Number | 48pt | Bold | Weight/reps input display |
| Large Title | 34pt | Bold | Screen titles |
| Title 1 | 28pt | Bold | Section headers |
| Title 2 | 22pt | Semibold | Card titles, exercise names |
| Title 3 | 20pt | Semibold | List item titles |
| Headline | 17pt | Semibold | Emphasized body |
| Body | 17pt | Regular | Standard text |
| Callout | 16pt | Regular | AI explanations |
| Subhead | 15pt | Regular | Supporting text |
| Footnote | 13pt | Regular | Timestamps, metadata |
| Caption | 12pt | Medium, Uppercase | Labels, section headers |
| Caption 2 | 11pt | Regular | Smallest text |

### 20.3 Spacing

| Token | Value | Usage |
|-------|-------|-------|
| xxs | 4pt | Tight spacing |
| xs | 8pt | Icon padding |
| sm | 12pt | Compact elements |
| md | 16pt | Standard padding |
| lg | 24pt | Section spacing |
| xl | 32pt | Major sections |
| xxl | 48pt | Screen margins |

### 20.4 Component Specifications

#### AI Suggestion Card

| Property | Value |
|----------|-------|
| Background | #2C2C2E with subtle gradient |
| Border | 1pt #64D2FF (AI cyan) left border |
| Border Radius | 12pt |
| Padding | 16pt |
| Icon | Wand/sparkle icon in AI cyan |
| Confidence Dot | 8pt circle (green/yellow/grey) |

#### Weight Suggestion Display

| Property | Value |
|----------|-------|
| Weight Display | 48pt Bold, centered |
| Confidence Indicator | Small dot + percentage |
| Explanation | 14pt Regular, grey, 1 line |
| Expand Button | Chevron to show details |
| Override Buttons | +5 / -5 pill buttons |

#### Confidence Indicator

| Level | Color | Icon |
|-------|-------|------|
| High (≥80%) | Green #30D158 | Filled circle |
| Medium (60-79%) | Yellow #FFD60A | Half-filled circle |
| Low (40-59%) | Grey #636366 | Empty circle |

#### Coach Alert Banner

| Property | Value |
|----------|-------|
| Height | 56pt minimum |
| Background | Varies by urgency (blue/yellow/red) |
| Icon | Left-aligned, 24pt |
| Title | 17pt Semibold |
| Dismiss | X button right-aligned |
| Action | Optional button |

### 20.5 Screen Layouts

#### Home Screen (Updated)

```
┌─────────────────────────────────────────┐
│ Good morning, Alex          ⚙️ Settings │
├─────────────────────────────────────────┤
│ ┌─────────────────────────────────────┐ │
│ │ 🤖 Coach Alert (if any)             │ │
│ │ "Consider a deload this week"    ✕  │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ NEXT WORKOUT                        │ │
│ │ Push Day A                          │ │
│ │                                     │ │
│ │ 6 exercises • 24 sets • ~55 min     │ │
│ │                                     │ │
│ │ 🤖 2 AI adjustments applied         │ │
│ │                                     │ │
│ │ • Bench Press         4 sets        │ │
│ │ • Incline DB Press    3 sets        │ │
│ │ • Cable Flye          3 sets  +1 🤖 │ │
│ │ • ...                                │ │
│ │                                     │ │
│ │ ┌─────────────────────────────────┐ │ │
│ │ │      🔴 START WORKOUT           │ │ │
│ │ └─────────────────────────────────┘ │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ○ Insert Rest Day                       │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ COMING UP                           │ │
│ │ ┌─────┐ ┌─────┐ ┌─────┐            │ │
│ │ │Pull │ │Legs │ │Rest │ →          │ │
│ │ │Day A│ │Day A│ │     │            │ │
│ │ └─────┘ └─────┘ └─────┘            │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ WEEKLY VOLUME          See All →    │ │
│ │ Chest ████████░░ 14/18 sets         │ │
│ │ Back  ██████████ 16/20 sets         │ │
│ │ ...                                  │ │
│ └─────────────────────────────────────┘ │
├─────────────────────────────────────────┤
│  🏠      📋       📊       ⚙️          │
│ Home   Program  History  Settings      │
└─────────────────────────────────────────┘
```

#### Active Workout Screen (Updated)

```
┌─────────────────────────────────────────┐
│ ✕  Push Day A        ✓ Saved    52: 30  │
├─────────────────────────────────────────┤
│                                         │
│ BENCH PRESS                             │
│ Set 2 of 4                    ● ● ○ ○   │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 🤖 AI Suggestion        87% confident│ │
│ │                                     │ │
│ │           185 lbs                   │ │
│ │                                     │ │
│ │ Small increase based on last week   │ │
│ │                          See why ▼  │ │
│ │                                     │ │
│ │    [-5]              [+5]           │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ SET 2 - Working                     │ │
│ │                                     │ │
│ │  Weight          Reps               │ │
│ │ ┌─────────┐    ┌─────────┐          │ │
│ │ │   185   │    │    8    │          │ │
│ │ │   lbs   │    │  reps   │          │ │
│ │ └─────────┘    └─────────┘          │ │
│ │                                     │ │
│ │ ┌─────────────────────────────────┐ │ │
│ │ │     ✓ MARK SET COMPLETE         │ │ │
│ │ └─────────────────────────────────┘ │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Set 1:  180 lbs × 8 ✓                    │
│ Set 3: ○ Pending                        │
│ Set 4: ○ Pending                        │
│                                         │
├─────────────────────────────────────────┤
│ NEXT:  Incline Dumbbell Press            │
│                              Skip Ex →  │
└─────────────────────────────────────────┘
```

#### AI Suggestion Expanded View

```
┌─────────────────────────────────────────┐
│ 🤖 Why 185 lbs?                      ✕  │
├─────────────────────────────────────────┤
│                                         │
│ Last Session                            │
│ ┌─────────────────────────────────────┐ │
│ │ 180 lbs × 8 reps @ RPE 7. 5          │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Factors Considered                      │
│                                         │
│ ↑ Last session performance              │
│   Hit top of rep range at moderate RPE  │
│                                         │
│ → Recovery status                       │
│   Normal recovery, no concerns          │
│                                         │
│ ↑ Progression trend                     │
│   Consistent progress over 4 weeks      │
│                                         │
│ → Fatigue level                         │
│   Moderate (0. 4), within normal range   │
│                                         │
│ Confidence:  87% (High)                  │
│ Source: AI Model (24 sessions of data)  │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │          Use 185 lbs                │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Or choose:   180 lbs  |  190 lbs         │
│                                         │
└─────────────────────────────────────────┘
```

#### Exercise Swap Modal

```
┌─────────────────────────────────────────┐
│ 🔄 Time for a change?                 ✕  │
├─────────────────────────────────────────┤
│                                         │
│ Barbell Row progress has stalled        │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ 📊 What we noticed:                  │ │
│ │                                     │ │
│ │ • 4 weeks at same weight            │ │
│ │ • RPE increased by 0.8              │ │
│ │ • 16 total sessions logged          │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Recommended Alternatives                │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ ⭐ Pendlay Row                       │ │
│ │ 92% match • SFR:  1.4 • Tried before │ │
│ │ Similar pattern, different curve    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ Chest-Supported Row                 │ │
│ │ 85% match • SFR: 1.6 • New          │ │
│ │ Removes lower back fatigue          │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ Seated Cable Row                    │ │
│ │ 78% match • SFR: 1.3 • New          │ │
│ │ Constant tension throughout ROM     │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ┌─────────────────────────────────────┐ │
│ │ Keep Barbell Row for now            │ │
│ │ (We'll check again in 2 weeks)      │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ ○ Try a deload instead                  │
│ ○ Show more alternatives                │
│                                         │
└─────────────────────────────────────────┘
```

---

## 21. Accessibility Requirements

### 21.1 Screen Reader Support

| Element | Accessibility Label Format |
|---------|---------------------------|
| Set Card | "Set [number], [weight] [unit], [reps] reps, [state]" |
| Weight Input | "Weight in [unit], current value [value]" |
| Reps Input | "Repetitions, current value [value]" |
| Complete Button | "Mark set complete" with hint "Double tap to complete this set" |
| AI Suggestion | "AI suggested weight:  [weight] [unit], [confidence] confidence" |
| Confidence Indicator | "[confidence level] confidence, [percentage] percent" |
| RPE Slider | "RPE rating, current value [value]" with hint "Adjust effort level from 1 to 10" |
| Rest Timer | "Rest timer, [time] remaining" |
| Coach Alert | "[urgency] alert:  [message]" |
| Volume Bar | "[muscle] volume, [current] of [target] sets" |

### 21.2 Announcements

| Event | Announcement |
|-------|--------------|
| Set completed | "Set [number] completed.  Moving to next set." |
| Exercise completed | "[Exercise] complete. [X] of [Y] exercises done." |
| Rest timer complete | "Rest complete. Ready for next set." |
| Workout completed | "Workout complete. [X] sets, [Y] total volume." |
| AI suggestion shown | "AI suggests [weight] [unit] with [confidence] confidence" |
| Coach alert appeared | "[urgency]:  [message]" |
| Exercise swap suggested | "Exercise swap suggested for [exercise]" |
| Deload recommended | "Deload week recommended" |

### 21.3 Motor Accessibility

| Requirement | Implementation |
|-------------|----------------|
| Touch targets | Minimum 44×44pt for all interactive elements |
| Gesture alternatives | All swipe actions have button alternatives |
| Timeout extensions | Rest timer can be extended indefinitely |
| Error prevention | Destructive actions require confirmation |
| AI suggestion override | Large +/- buttons, not just text input |

### 21.4 Cognitive Accessibility

| Requirement | Implementation |
|-------------|----------------|
| Clear language | AI explanations in plain English |
| Consistent layout | Same screen structure throughout app |
| Progress indication | Clear indication of workout progress |
| Undo support | Can edit completed sets |
| Confirmation | Destructive actions require confirmation |

---

## 22. Data Backup & Recovery

### 22.1 Backup Strategy

| Type | Trigger | Retention |
|------|---------|-----------|
| Auto Backup | Every 7 days | Last 5 backups |
| Manual Backup | User-initiated | Until manually deleted |
| Pre-Restore Backup | Before any restore operation | 1 backup |
| Pre-Update Backup | Before app updates | 1 backup |

### 22.2 Backup Contents

| Data | Included | Notes |
|------|----------|-------|
| User Profile | ✓ | All profile data |
| Programs | ✓ | Including AI-generated |
| Program Days | ✓ | |
| Custom Exercises | ✓ | |
| Workout Templates | ✓ | |
| Workout Sessions | ✓ | Full history |
| Logged Sets | ✓ | |
| Exercise Feedback | ✓ | |
| User Exercise Metrics | ✓ | SFR data |
| User Volume Landmarks | ✓ | |
| Fatigue Logs | ✓ | |
| Weekly Check-ins | ✓ | |
| Deload History | ✓ | |
| Exercise Swap History | ✓ | |
| AI Recommendations Log | ✓ | |
| User Preferences | ✓ | |
| ML Personalization Data | ✓ | Model weights |
| Built-in Exercises | ✗ | Bundled with app |
| Base ML Models | ✗ | Bundled with app |

### 22.3 Backup Format

| Property | Value |
|----------|-------|
| Format | JSON (encrypted) |
| Encryption | AES-256-GCM with user password |
| Key Derivation | PBKDF2 with SHA-256, 100,000 iterations |
| Compression | Optional gzip for large backups |
| Naming | apex-backup-YYYY-MM-DDTHH-mm-ss.apex |
| Max Size | ~100MB for 5 years of data with full AI history |

### 22.4 Recovery Options

| Scenario | Recovery Method |
|----------|-----------------|
| Corrupted database | Attempt SQLite repair, then restore from backup |
| Accidental deletion | Restore from backup |
| New device | Export backup, transfer file, import on new device |
| App reinstall | Import from previously exported backup file |
| ML personalization lost | Rebuilds automatically from workout history |

---

## 23. Implementation Phases

### Phase 1: Foundation (Weeks 1-4)

| Deliverable | Success Criteria |
|-------------|------------------|
| Project setup | Builds on iOS 15+ and Android 9+ |
| SQLite database (expanded schema) | All tables created, migrations run |
| Redux store (with AI slices) | State persists across restarts |
| Design system | Colors, typography, spacing, base components |
| Navigation | Tab bar and stack navigators functional |
| Error handling | Sentry integration, error boundaries |
| Base ML model loading | Models load within 2 seconds |

### Phase 2: User Profile & Onboarding (Weeks 5-8)

| Deliverable | Success Criteria |
|-------------|------------------|
| Onboarding flow | All 8 steps functional |
| User profile storage | Profile persists correctly |
| Equipment selection | Filters exercises correctly |
| Injury recording | Affects exercise suggestions |
| Preference management | Favorites/dislikes respected |
| Profile editing | All fields editable from settings |

### Phase 3: Core Workout + AI Suggestions (Weeks 9-14)

| Deliverable | Success Criteria |
|-------------|------------------|
| Set card component | All states render correctly |
| Numeric inputs | Validation, large touch targets |
| Active workout screen | Full layout with AI suggestions |
| Weight prediction model | Inference < 100ms |
| AI suggestion display | Confidence, explanation, override |
| Auto-save system | Saves complete in < 50ms |
| Set completion flow | State machine transitions correctly |
| RPE modal | Appears after last set, saves data |
| Rest timer | Starts on completion, haptics work |
| Draft system | 12-hour expiration, auto-complete |
| Draft recovery | Notification appears, resume works |

### Phase 4: SFR & Exercise Intelligence (Weeks 15-18)

| Deliverable | Success Criteria |
|-------------|------------------|
| SFR calculation engine | Calculates for all exercises |
| Base SFR data | All library exercises have base scores |
| User SFR personalization | Updates after each session |
| Exercise ranking by SFR | Sorted lists in selection |
| Pain penalty system | Pain affects SFR correctly |
| Muscle connection tracking | Affects stimulus modifier |

### Phase 5: Volume Management (Weeks 19-22)

| Deliverable | Success Criteria |
|-------------|------------------|
| Volume landmark calculation | Personalized per muscle |
| Weekly volume tracking | Accurate set counting |
| Volume status display | Visual in analytics |
| Volume optimization model | Suggests adjustments |
| Auto-adjustment suggestions | User can accept/reject |
| Volume history | Trends visible in analytics |

### Phase 6: Exercise Swap & Deload (Weeks 23-28)

| Deliverable | Success Criteria |
|-------------|------------------|
| Plateau detection model | Identifies stalls accurately |
| Swap trigger system | Detects all trigger conditions |
| Bad week detection | Suppresses false swap triggers |
| Alternative ranking | Top 3 alternatives by SFR |
| Swap flow UI | Modal with explanations |
| Fatigue estimation model | Predicts fatigue accurately |
| Proactive deload suggestions | Warns before plateau |
| Deload protocol application | Adjusts workout weights |
| Deload effectiveness tracking | Measures recovery |

### Phase 7: Program Generation (Weeks 29-34)

| Deliverable | Success Criteria |
|-------------|------------------|
| Split selection logic | Appropriate for user |
| Exercise selection algorithm | Respects all constraints |
| Volume distribution | Within landmarks |
| Program generation UI | < 5 seconds generation |
| Program rationale | Explains all decisions |
| Program modification | Edit any aspect |
| Template programs | 10+ available |

### Phase 8: Real-Time Adaptation & Polish (Weeks 35-40)

| Deliverable | Success Criteria |
|-------------|------------------|
| Bad day detection | Triggers after 3 sets |
| Real-time adaptations | Non-intrusive suggestions |
| Weekly check-in | Prompts correctly |
| Coach dashboard | Home screen integration |
| Backup/restore (full) | All AI data included |
| Auto-backup | Weekly automatic |
| History & analytics | Full AI insights |
| Performance optimization | All targets met |

### Phase 9: Quality & Launch (Weeks 41-46)

| Deliverable | Success Criteria |
|-------------|------------------|
| Accessibility audit | All screens pass |
| ML model validation | All accuracy targets met |
| Comprehensive testing | 80% coverage |
| Beta testing | 50+ users, critical bugs fixed |
| App store submission | Listings approved |
| Documentation | User guide complete |

---

## 24. Testing Requirements

### 24.1 Coverage Targets

| Test Type | Coverage Target |
|-----------|-----------------|
| Unit Tests | ≥ 80% |
| Integration Tests | Key user flows |
| E2E Tests | Critical paths |
| ML Model Tests | All models validated |
| Accessibility Tests | All screens |
| Performance Tests | All targets verified |

### 24.2 Critical Test Scenarios

#### Workout Flow with AI

- [ ] Start workout, AI suggestions displayed
- [ ] Override AI suggestion, override recorded
- [ ] Complete sets with AI weight, performance tracked
- [ ] AI adapts to performance mid-workout
- [ ] Complete exercise with RPE feedback
- [ ] RPE affects next suggestion
- [ ] Finish workout, AI metrics updated
- [ ] Verify history entry with AI data

#### SFR System

- [ ] SFR calculates with base data only
- [ ] SFR updates after session
- [ ] Pain report affects SFR
- [ ] Low muscle connection affects SFR
- [ ] Exercise ranking reflects SFR
- [ ] SFR confidence increases with data

#### Volume Management

- [ ] Volume tracks correctly per muscle
- [ ] Compound exercises count for multiple muscles
- [ ] Landmarks personalize over time
- [ ] Below MEV triggers suggestion
- [ ] Above MRV triggers warning
- [ ] Volume adjustments apply correctly

#### Exercise Swapping

- [ ] Plateau detected after 4 weeks
- [ ] Bad week suppresses swap
- [ ] Pain triggers immediate swap option
- [ ] Alternatives ranked by SFR
- [ ] Favorites protected from auto-swap
- [ ] Swap decision logged

#### Deload System

- [ ] Fatigue calculated correctly
- [ ] Proactive deload suggested before plateau
- [ ] Deload protocol adjusts weights
- [ ] Deload effectiveness measured
- [ ] Deload history tracked

#### Program Generation

- [ ] Generated program respects constraints
- [ ] Favorites included
- [ ] Injuries avoided
- [ ] Volume within landmarks
- [ ] Rationale explains decisions
- [ ] User can edit generated program

#### ML Fallbacks

- [ ] Weight prediction falls back gracefully
- [ ] SFR uses base values on failure
- [ ] Volume uses defaults on failure
- [ ] No user-facing errors from ML

### 24.3 ML Model Validation

| Model | Test | Target |
|-------|------|--------|
| Weight Prediction | MAE on test set | < 5 lbs |
| Weight Prediction | Within 10% accuracy | ≥ 95% |
| Plateau Detection | Recall | ≥ 85% |
| Plateau Detection | Precision | ≥ 80% |
| Fatigue Estimation | Correlation | ≥ 0.75 |
| Volume Optimization | User acceptance | ≥ 70% |

### 24.4 Performance Benchmarks

| Scenario | Measurement | Target |
|----------|-------------|--------|
| Cold start with ML | Time to interactive | < 3s |
| Weight suggestion | Generation time | < 100ms |
| SFR calculation | Per exercise | < 50ms |
| Full session analysis | Post-workout | < 500ms |
| Program generation | Full program | < 5s |
| Memory (1 hour session) | Peak heap size | < 350MB |
| Battery (1 hour session) | Drain percentage | < 5% |

---

## 25. Success Metrics

### 25.1 Key Performance Indicators

| Metric | Target | Measurement |
|--------|--------|-------------|
| Crash-Free Rate | ≥ 99. 9% | Sentry |
| ANR Rate | < 0.1% | Play Console / App Store Connect |
| D1 Retention | ≥ 60% | Analytics |
| D7 Retention | ≥ 40% | Analytics |
| D30 Retention | ≥ 25% | Analytics |
| Workouts/Week/User | ≥ 3. 5 | Analytics |
| AI Suggestion Acceptance | ≥ 70% | Analytics |
| AI Override Rate | < 30% | Analytics |
| Swap Suggestion Acceptance | ≥ 50% | Analytics |
| Deload Suggestion Acceptance | ≥ 60% | Analytics |
| App Store Rating | ≥ 4.5 | App Store / Play Store |
| Onboarding Completion | ≥ 85% | Analytics |
| Weekly Check-in Completion | ≥ 60% | Analytics |

### 25.2 AI Coach Quality Metrics

| Metric | Target | Measurement |
|--------|--------|-------------|
| Weight Prediction Accuracy (within 5%) | ≥ 80% | Comparison to actual |
| Plateau Detection Accuracy | ≥ 80% | User confirmation |
| Deload Timing Satisfaction | ≥ 75% | Post-deload survey |
| Exercise Swap Success | ≥ 70% | Progress after swap |
| Program Satisfaction | ≥ 80% | User rating |
| Explanation Clarity | ≥ 85% | User feedback |

### 25.3 Quality Gates (Pre-Release)

| Gate | Requirement |
|------|-------------|
| Test Coverage | ≥ 80% |
| Lint/Type Errors | 0 |
| All Tests Passing | 100% |
| Performance Targets | All met |
| ML Accuracy Targets | All met |
| Accessibility Audit | All screens pass |
| Beta Feedback | No critical bugs |

---

## 26. Deployment Checklist

### 26.1 Pre-Submission

- [ ] All quality gates passing
- [ ] No console. log or debug flags in production
- [ ] Sentry configured for production environment
- [ ] ML models optimized and bundled
- [ ] App icons and screenshots prepared
- [ ] App description and keywords finalized
- [ ] Privacy policy published (emphasizing offline-only, no data collection)
- [ ] Support email configured

### 26.2 App Store Assets

| Asset | iOS Requirement | Android Requirement |
|-------|-----------------|---------------------|
| App Icon | 1024×1024 | 512×512 |
| Screenshots | 6. 7", 6.5", 5.5" iPhones | Phone + Tablet |
| Feature Graphic | - | 1024×500 |
| Description | 4000 chars max | 4000 chars max |
| Short Description | - | 80 chars |
| Keywords | 100 chars | - |
| Privacy Label | Required | Data safety section |

### 26.3 Post-Launch Monitoring

| Metric | Alert Threshold | Action |
|--------|-----------------|--------|
| Crash Rate | > 0.5% | Investigate immediately |
| ANR Rate | > 0.2% | Investigate within 24 hours |
| ML Error Rate | > 1% | Check fallback system |
| 1-star Reviews | Any mentioning AI | Respond within 48 hours |
| Support Tickets | > 10/day | Review common issues |

---

## 27. Appendices

### Appendix A:  Pre-built Programs

| Program | Level | Days | Description | AI Enhancement |
|---------|-------|------|-------------|----------------|
| Starting Strength | Beginner | 3 | A/B alternating, compound focus | Conservative progression |
| StrongLifts 5×5 | Beginner | 3 | A/B alternating, 5×5 structure | Linear progression tracking |
| Push Pull Legs | Intermediate | 6 | PPL split, high frequency | Volume auto-regulation |
| Upper/Lower | Intermediate | 4 | Strength + hypertrophy days | SFR-optimized selection |
| PHUL | Intermediate | 4 | Power + hypertrophy upper/lower | Dual-focus periodization |
| PHAT | Advanced | 5 | Power + hypertrophy combination | Advanced fatigue management |
| nSuns 5/3/1 | Intermediate | 4-6 | High volume 5/3/1 variant | Volume landmark integration |
| Wendler 5/3/1 | Intermediate | 4 | Classic 5/3/1 progression | Deload week automation |
| GZCLP | Beginner | 3-4 | Tiered exercise approach | Tier progression tracking |
| Bro Split | Any | 5 | Traditional bodypart split | Per-muscle optimization |
| AI Optimized | Any | 2-7 | Fully AI-generated | Maximum personalization |

### Appendix B: Exercise Library Categories

| Category | Count | Examples |
|----------|-------|----------|
| Chest | 20+ | Bench Press, Incline DB Press, Cable Flye, Dips |
| Back (General) | 25+ | Deadlift, Pull-up, Rows, Pullovers |
| Lats | 15+ | Lat Pulldown, Pull-up, Straight-arm Pulldown |
| Traps | 8+ | Shrugs, Face Pulls, Rack Pulls |
| Side Delts | 10+ | Lateral Raise, Upright Row, Cable Lateral |
| Front Delts | 6+ | Front Raise, Overhead Press (secondary) |
| Rear Delts | 10+ | Reverse Flye, Face Pull, Rear Delt Row |
| Biceps | 12+ | Barbell Curl, Hammer Curl, Preacher Curl |
| Triceps | 12+ | Close Grip Bench, Pushdown, Skull Crusher |
| Forearms | 6+ | Wrist Curl, Reverse Curl, Farmer's Walk |
| Quads | 15+ | Squat, Leg Press, Lunge, Leg Extension |
| Hamstrings | 10+ | RDL, Leg Curl, Good Morning |
| Glutes | 8+ | Hip Thrust, Glute Bridge, Cable Pull-through |
| Calves | 6+ | Standing Calf Raise, Seated Calf Raise |
| Abs | 12+ | Crunch, Hanging Leg Raise, Plank, Cable Crunch |
| Obliques | 6+ | Side Bend, Woodchop, Russian Twist |
| **Total** | **175+** | |

### Appendix C: Movement Patterns

| Pattern | Description | Example Exercises |
|---------|-------------|-------------------|
| Horizontal Push | Pushing away from body horizontally | Bench Press, Push-up |
| Horizontal Pull | Pulling toward body horizontally | Barbell Row, Cable Row |
| Vertical Push | Pushing overhead | Overhead Press, Arnold Press |
| Vertical Pull | Pulling from overhead | Pull-up, Lat Pulldown |
| Hip Hinge | Bending at hips with neutral spine | Deadlift, RDL, Good Morning |
| Squat | Bending at knees and hips together | Back Squat, Front Squat, Leg Press |
| Lunge | Single-leg squat pattern | Walking Lunge, Bulgarian Split Squat |
| Isolation Curl | Elbow flexion | Bicep Curl, Hammer Curl |
| Isolation Extension | Elbow extension | Tricep Pushdown, Skull Crusher |
| Isolation Raise | Shoulder abduction/flexion | Lateral Raise, Front Raise |
| Isolation Fly | Shoulder horizontal adduction | Cable Fly, Pec Deck |
| Carry | Loaded walking | Farmer's Walk, Suitcase Carry |
| Rotation | Rotational core movement | Woodchop, Russian Twist |

### Appendix D: SFR Base Scores (Sample)

| Exercise | Base Stimulus | Base Fatigue | Base SFR | Joint Stress |
|----------|---------------|--------------|----------|--------------|
| Bench Press | 8 | 6 | 1.33 | 4 |
| Incline DB Press | 8 | 5 | 1.60 | 3 |
| Cable Fly | 7 | 3 | 2.33 | 2 |
| Barbell Row | 8 | 7 | 1.14 | 5 |
| Chest-Supported Row | 7 | 4 | 1.75 | 2 |
| Lat Pulldown | 7 | 4 | 1.75 | 2 |
| Pull-up | 8 | 5 | 1.60 | 3 |
| Overhead Press | 7 | 6 | 1.17 | 5 |
| Lateral Raise | 7 | 2 | 3.50 | 2 |
| Barbell Curl | 6 | 4 | 1.50 | 3 |
| Cable Curl | 6 | 2 | 3.00 | 1 |
| Squat | 9 | 8 | 1.13 | 6 |
| Leg Press | 8 | 5 | 1.60 | 3 |
| Leg Extension | 6 | 3 | 2.00 | 4 |
| RDL | 8 | 7 | 1.14 | 5 |
| Leg Curl | 6 | 2 | 3.00 | 1 |
| Hip Thrust | 8 | 4 | 2.00 | 2 |

### Appendix E: Default Volume Landmarks

| Muscle Group | MV | MEV | MAV | MRV | Notes |
|--------------|-----|-----|-----|-----|-------|
| Chest | 6 | 10 | 18 | 22 | Responds well to frequency |
| Back (overall) | 6 | 10 | 20 | 25 | Can handle high volume |
| Lats | 4 | 8 | 16 | 20 | Subset of back |
| Traps | 0 | 4 | 12 | 16 | Often hit by other exercises |
| Side Delts | 4 | 8 | 18 | 25 | Very high MRV |
| Front Delts | 0 | 0 | 8 | 12 | Hit by pressing |
| Rear Delts | 0 | 6 | 14 | 18 | Often neglected |
| Biceps | 4 | 8 | 16 | 20 | Recovers quickly |
| Triceps | 4 | 6 | 14 | 18 | Hit by pressing |
| Forearms | 0 | 4 | 12 | 16 | Often hit by gripping |
| Quads | 6 | 8 | 16 | 20 | Taxing to recover from |
| Hamstrings | 4 | 6 | 14 | 18 | Often undertrained |
| Glutes | 0 | 4 | 12 | 16 | Hit by squats/hinges |
| Calves | 4 | 8 | 16 | 20 | Stubborn, needs frequency |
| Abs | 0 | 6 | 16 | 20 | Recovers quickly |
| Obliques | 0 | 4 | 12 | 16 | Often hit by compounds |

### Appendix F:  Fatigue Indicator Calculations

```typescript
// RPE Creep:  Compare recent RPE to historical at same weight
RPE_Creep = (avg_rpe_last_2_weeks - avg_rpe_previous_4_weeks) / 2
// Normalized to 0-1 range

// Performance Decline: Compare recent performance to expected
Performance_Decline = 1 - (actual_volume_last_2_weeks / expected_volume)
// Capped at 0-1 range

// Recovery Issues: Based on rest time trends and session completion
Recovery_Issues = (avg_rest_time_increase / baseline_rest) * 0.5 +
                  (incomplete_sessions / total_sessions) * 0.5
// Normalized to 0-1 range

// Motivation Drop: Based on skipped workouts and session duration
Motivation_Drop = (skipped_workouts / scheduled_workouts) * 0.6 +
                  (1 - avg_session_duration / target_duration) * 0.4
// Normalized to 0-1 range

// Pain Frequency: Based on pain reports
Pain_Frequency = pain_reports_last_2_weeks / total_exercises_last_2_weeks
// Normalized to 0-1 range
```

### Appendix G: API Reference (Internal Services)

```typescript
// Core AI Coach Service Methods
interface AICoachService {
  // Weight Suggestions
  getWeightSuggestion(userId: string, exerciseId: string): Promise<WeightSuggestion>;
  recordWeightOverride(suggestionId: string, actualWeight: number): Promise<void>;
  
  // SFR Management
  calculateSFR(userId: string, exerciseId: string): Promise<SFRProfile>;
  getRankedExercises(userId: string, muscleGroup: MuscleGroup): Promise<ExerciseSFRProfile[]>;
  
  // Volume Management
  getVolumeLandmarks(userId: string, muscleGroup: MuscleGroup): Promise<VolumeLandmarks>;
  getVolumeAnalysis(userId: string): Promise<VolumeAnalysis>;
  suggestVolumeAdjustments(userId: string): Promise<VolumeAdjustment[]>;
  
  // Exercise Swapping
  checkSwapTriggers(userId: string, exerciseId: string): Promise<SwapRecommendation | null>;
  getAlternatives(userId: string, exerciseId: string, reason: SwapReason): Promise<ExerciseAlternative[]>;
  recordSwapDecision(swapId: string, decision: SwapDecision): Promise<void>;
  
  // Deload Management
  predictDeloadNeed(userId: string): Promise<DeloadPrediction>;
  generateDeloadProtocol(userId: string, prediction: DeloadPrediction): Promise<DeloadProtocol>;
  startDeload(userId: string, protocol: DeloadProtocol): Promise<void>;
  
  // Program Generation
  generateProgram(userId: string, options?:  ProgramOptions): Promise<GeneratedProgram>;
  
  // Real-Time Adaptation
  analyzeCurrentSession(sessionId: string): Promise<SessionAdaptation[]>;
  applyAdaptation(adaptationId: string): Promise<void>;
  dismissAdaptation(adaptationId:  string): Promise<void>;
  
  // Recommendations
  getRecommendations(userId: string): Promise<CoachRecommendations>;
}
```

---

## Document Sign-off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Product Owner | | | |
| Tech Lead | | | |
| AI/ML Lead | | | |
| UX Designer | | | |
| QA Lead | | | |

---

**END OF REQUIREMENTS DOCUMENT v3.0**

*This document is a living specification.  All changes must be reviewed and approved before implementation.  Update the version number and changelog for any modifications.*

*Major changes in v3.0 reflect a fundamental shift from basic ML weight suggestions to a comprehensive AI coaching system that manages all aspects of hypertrophy training.*
````

---

## Summary of Major Changes (v2.1 → v3.0)

| Section | Changes |
|---------|---------|
| **Project Vision** | Added AI Coach as primary differentiator, expanded design philosophy |
| **Glossary** | Added SFR, MEV, MAV, MRV, volume landmarks, fatigue terminology |
| **Technical Stack** | Increased RAM requirement, added background tasks |
| **Performance Targets** | Added ML-specific inference targets |
| **Data Model** | 12 new tables for AI/ML system, expanded exercise schema |
| **State Management** | Added coach, sfr, volume, fatigue slices |
| **NEW:  User Profile System** | Complete section with onboarding flow |
| **Core Features** | Updated workout logging with AI integration, added weekly check-in |
| **NEW: Hypertrophy AI Coach** | Complete system overview and capabilities |
| **NEW: SFR Engine** | Full specification for stimulus-to-fatigue system |
| **NEW: Volume Management** | Personalized volume landmarks and optimization |
| **NEW: Exercise Swap Engine** | Intelligent exercise selection and swapping |
| **NEW:  Deload System** | Proactive fatigue prediction and deload scheduling |
| **NEW: Program Generation** | AI-powered personalized program creation |
| **NEW: Real-Time Adaptation** | Mid-workout intelligence and adjustments |
| **NEW: ML Architecture** | Multi-model system with 6 specialized models |
| **UI/UX** | Updated screens with AI integration, new components |
| **Implementation Phases** | Expanded from 6 to 9 phases |
| **Testing** | Added ML model validation requirements |
| **Success Metrics** | Added AI-specific quality metrics |
| **Appendices** | Added SFR scores, volume landmarks, fatigue calculations |

This document now serves as a complete specification for building an AI-powered hypertrophy coaching system. 
