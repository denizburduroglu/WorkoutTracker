# SPEC-001: Workout Template Models

**Status**: Draft
**Created**: 2025-10-30
**Updated**: 2025-10-30
**Author**: Project Team

## Overview

Define the core domain models for creating and managing workout templates. Users can build reusable workout routines focused on dumbbell exercises and stretches. Each exercise in a template can specify either sets/reps/weight OR sets/duration.

## Goals

- Create a simple system for building workout templates
- Support two exercise categories for organization: strength and stretching
- Allow flexible data entry: each template exercise can use reps+weight OR duration
- Enable users to organize exercises into reusable workout routines

## Non-Goals

- Workout logging/history tracking - future spec
- Progress tracking over time - future spec
- Cardio exercises - not needed for current use case
- Social features - future spec
- Mobile app - future spec

## Requirements

### Functional Requirements

1. **FR-1**: The system shall store workout templates with a name and optional description
2. **FR-2**: The system shall categorize exercises as either "strength" or "stretching"
3. **FR-3**: The system shall allow template entries with: sets, reps, and weight
4. **FR-4**: The system shall allow template entries with: sets and duration (time)
5. **FR-5**: Each template exercise entry uses EITHER reps+weight OR duration (not both)
6. **FR-6**: The system shall allow users to add multiple exercises to a template
7. **FR-7**: The system shall maintain a library of predefined exercises (dumbbell movements and stretches)
8. **FR-8**: The system shall allow users to create custom exercises

### Non-Functional Requirements

1. **NFR-1**: Models should use appropriate indexes for query performance
2. **NFR-2**: Data should be normalized to avoid redundancy
3. **NFR-3**: Templates should be easy to duplicate/modify

## Design

### Data Models

#### Exercise

An exercise definition (e.g., "Dumbbell Chest Press", "Hamstring Stretch")

**Note**: The category is for organization only - any exercise can be added to a template with either reps/weight or duration.

```python
class Exercise(models.Model):
    STRENGTH = 'strength'
    STRETCHING = 'stretching'

    CATEGORY_CHOICES = [
        (STRENGTH, 'Strength'),
        (STRETCHING, 'Stretching'),
    ]

    name = models.CharField(max_length=100)
    category = models.CharField(max_length=20, choices=CATEGORY_CHOICES)
    description = models.TextField(blank=True)
    muscle_groups = models.JSONField(default=list)  # e.g., ["chest", "triceps"]
    created_by = models.ForeignKey(User, on_delete=models.SET_NULL, null=True, blank=True)
    is_custom = models.BooleanField(default=False)  # True if user-created
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['name']
        indexes = [
            models.Index(fields=['category']),
            models.Index(fields=['created_by']),
        ]

    def __str__(self):
        return f"{self.name} ({self.get_category_display()})"
```

#### WorkoutTemplate

A reusable workout routine

```python
class WorkoutTemplate(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE, related_name='workout_templates')
    name = models.CharField(max_length=200)
    description = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        ordering = ['-updated_at']
        indexes = [
            models.Index(fields=['user', '-updated_at']),
        ]

    def __str__(self):
        return self.name
```

#### TemplateExercise

An exercise within a workout template.

**Each entry uses EITHER (sets + reps + weight) OR (sets + duration).**

```python
class TemplateExercise(models.Model):
    template = models.ForeignKey(WorkoutTemplate, on_delete=models.CASCADE, related_name='exercises')
    exercise = models.ForeignKey(Exercise, on_delete=models.PROTECT)
    order = models.PositiveIntegerField()  # Order within template

    # Always specified
    sets = models.PositiveIntegerField(default=1)

    # Option 1: For rep-based exercises (typically strength)
    reps = models.PositiveIntegerField(null=True, blank=True)
    weight = models.DecimalField(max_digits=6, decimal_places=2, null=True, blank=True)  # in lbs

    # Option 2: For duration-based exercises (typically stretching)
    duration_seconds = models.PositiveIntegerField(null=True, blank=True)

    notes = models.TextField(blank=True)
    created_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        ordering = ['order']
        indexes = [
            models.Index(fields=['template', 'order']),
        ]

    def __str__(self):
        return f"{self.exercise.name} in {self.template.name}"

    def clean(self):
        """Validate that either reps+weight OR duration is specified, not both."""
        from django.core.exceptions import ValidationError

        has_reps_data = self.reps is not None or self.weight is not None
        has_duration_data = self.duration_seconds is not None

        if has_reps_data and has_duration_data:
            raise ValidationError("Specify either reps/weight OR duration, not both.")

        if not has_reps_data and not has_duration_data:
            raise ValidationError("Must specify either reps/weight OR duration.")
```

## User Stories

### Story 1: Create Workout Template

**As a** user
**I want** to create workout templates with my preferred exercises
**So that** I can reuse my favorite workout routines

#### Acceptance Criteria

- [ ] Can create a template with a name and description
- [ ] Can add exercises with sets + reps + weight
- [ ] Can add exercises with sets + duration
- [ ] Can reorder exercises within a template
- [ ] Can save and view my templates

### Story 2: Exercise Library

**As a** user
**I want** to choose from predefined dumbbell exercises and stretches
**So that** I don't have to manually enter exercise names each time

#### Acceptance Criteria

- [ ] Can browse available exercises by category (strength/stretching)
- [ ] Can view exercise descriptions and muscle groups
- [ ] Can create custom exercises if needed
- [ ] Predefined exercises include common dumbbell movements and stretches

## Edge Cases

1. **Empty templates**: User creates template but doesn't add exercises yet - should be allowed
2. **Custom exercises**: Users should be able to create their own exercises beyond predefined ones
3. **Units**: Use pounds (lbs) for weight as default for dumbbell workouts
4. **Mixed template**: A single template can have both rep-based and duration-based exercises
5. **Reordering exercises**: User should be able to change exercise order after creation
6. **Deleting exercises from templates**: Shouldn't affect the exercise library, just remove from template
7. **Validation**: System should prevent saving a template exercise with both reps/weight AND duration specified

## Dependencies

- Django 5.2+
- User authentication (Django built-in)

## Testing Strategy

- Unit tests for model methods and properties
- Test model relationships and cascading deletes
- Test data validation (negative reps, negative weight, invalid durations)
- Test validation that prevents both reps+weight and duration
- Test validation that requires at least one data type
- Test ordering of exercises within templates
- Test custom vs predefined exercises
- Test query performance with indexes

## Security Considerations

- Users should only access their own workout templates
- Users can view predefined exercises, but only modify their own custom exercises
- Validate all numeric inputs (prevent negative values, reasonable maximums)
- Sanitize text inputs in names, descriptions, and notes to prevent injection attacks

## Migration Strategy

- Initial migration, no backward compatibility needed
- Prepopulate database with common dumbbell exercises and stretches
- Consider adding a management command for seeding exercise data

## Open Questions

- [ ] Should we prepopulate common dumbbell exercises on first run, or require manual import?
- [ ] Should templates be duplicatable/copyable for creating variations?
- [ ] Do we need rest time between sets in the template?
- [ ] Should stretches support "per side" notation (e.g., 30 seconds per leg)?

## Example Data

### Predefined Strength Exercises (Dumbbell)
Day 1: Upper body 
- Bent Over Dumbbell Row
  - 4 sets 8-10 Reps 22.5pounds
- Dumbbell Bench Press 
  - 4 sets 8-10 Reps 30
- Dumbbell Lateral Raise
  - 3 sets 8-10 Reps 17.5pounds
- Dumbbell Pullover
  - 3 sets 8-12 reps 30pounds
- Dumbbell Bicep Curl
  - 2 sets 8-12 reps 22.5pounds
- Dumbbell Tricep Extension
  - 2 sets 8-12 reps 30pounds
- Dumbbell Shrug
  - 2 sets 12-15 reps 35pounds
Day 2: Lower Body
- Goblet Squat
  - 4 sets 8-10 reps 30pounds
- Dumbbell Stiff Leg Deadlift
  - 4 sets 8-10 reps 30pounds
- Dumbbell Sumo Squat
  - 3 sets 8-12 reps 30pounds
- Dumbbell Swings
  - 3 sets 8-12 reps 30pounds
- Dumbbell Calf Raises
  - 3 sets 8-12 reps 35pounds
- Plank
  - 3 sets 30seconds
Day 3: Upper Body
- One Arm Dumbbell Row
  - 4 sets 8-10 reps 22.5pounds, each side
- Dumbbell Shoulder Press 
  - 4 sets 8-10 reps 22.5pounds
- Incline Dumbbell Bench Press (45degrees)
  - 3 sets 8-12 reps 30 pounds
- Chest Supported Dumbbell Row (45degrees)
  - 3 sets 8-12 reps 22.5pounds
- Dumbbell Hammer Curl
  - 2 sets 8-12 reps 22.5pounds
- Dumbbell Chest press
  - 2 sets 8-12 reps 30 pounds
- Seated Dumbbell Shrug
  - 2 sets 12-15 reps 35pounds

Day 4: Lower Body
- Dumbbell Stiff Leg Deadlift
  - 4 sets 8-10 reps 30pounds
- Dumbbell Rear Lunge
  - 4 sets 8-10 reps 30pounds, each side
- Dumbbell Hip Thrust
  - 4 sets 8-10 reps 30pounds
- Dumbbell Split Squat (one leg on bench)
  - 3 sets 8-12 reps 30pounds
- Dumbbell seated calf raise
  - 3 sets 8-12 reps 40pounds
- Plank
  - 3 sets 30seconds


### Predefined Stretching Exercises
- Lateral Neck Stretch
    - Should pull your head to one side while raising and lowering your opposite arm. 30s on each side
- Shoulder Reach
    - Should clasp your hands behind your back and push out your chest. 30s
- Child Pose
  - 30s
- Cat/Cow Pose
  - 10 times
- Twisting Pec Stretch
  - Either angle your arm 90 degrees or fully extended. Then while lying on your belly shift over your arm. 30s on each side
- Mckenzie Push Up
  - Ly on your belly and push your upper body up into a seal position. 10 reps
- Lying Knee Tuck
  - While on your back, tuck your knees and wrap your arms around your legs. 20s
- Lying Knee Drop
  - While on your back, extend your arms out to your side, drop your knees 90 degrees to one side. Switch sides and repeat 10 reps
- Squat to Pike
  - Squat down to your knees and then push your body up into a pike position. 20 reps
- Twisting Lunge Stretch
  - 30s each side
- Low Lunge
  - With your leg picked up lunge and rely on your arms to move around in this position. 20s each side
- Frog Rocks
  - 20s

### Example Template: "Upper Body & Stretch"
1. Dumbbell Chest Press - 3 sets x 12 reps @ 25 lbs
2. Dumbbell Shoulder Press - 3 sets x 10 reps @ 20 lbs
3. Dumbbell Bicep Curl - 3 sets x 12 reps @ 15 lbs
4. Shoulder Stretch - 2 sets x 30 seconds
5. Chest Stretch - 2 sets x 30 seconds

## References

- Similar apps: Strong, FitNotes, Hevy (but simpler - templates only)
- Django models best practices: https://docs.djangoproject.com/en/5.2/topics/db/models/
