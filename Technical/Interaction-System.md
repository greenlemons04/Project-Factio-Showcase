# Interaction System

Project Factio uses a reusable interaction system built with Unreal Engine 5 C++ and designed to work alongside Blueprints.

The player performs a forward trace from the first-person camera to determine whether the object being looked at is interactable. When the player presses the interaction input, the currently detected object receives an interaction event.

The base interactable actor exposes its interaction function as a `BlueprintNativeEvent`. This allows individual doors, pickups, puzzle objects, and other environmental objects to define their own behavior in Blueprint or C++ without changing the player class.

## Player Detection

```cpp
void AHorrorPC::TraceForInteractable()
{
    CurrentInteractable = nullptr;

    if (!FirstPersonCamera || !GetWorld())
    {
        return;
    }

    FVector Start = FirstPersonCamera->GetComponentLocation();
    FVector End = Start + FirstPersonCamera->GetForwardVector() * InteractDistance;

    FHitResult HitResult;

    FCollisionQueryParams QueryParams;
    QueryParams.AddIgnoredActor(this);

    bool bHit = GetWorld()->LineTraceSingleByChannel(
        HitResult,
        Start,
        End,
        ECC_Visibility,
        QueryParams
    );

    if (bHit)
    {
        CurrentInteractable = Cast<AInteractableActor>(HitResult.GetActor());
    }
}
```

## Interaction

```cpp
void AHorrorPC::Interact()
{
    if (CurrentInteractable)
    {
        CurrentInteractable->Interact(this);
    }
}
```

## Reusable Interactable Base

```cpp
UFUNCTION(BlueprintNativeEvent, BlueprintCallable, Category = "Interaction")
void Interact(AHorrorPC* Player);

virtual void Interact_Implementation(AHorrorPC* Player);
```

## Design Goal

The system separates interaction detection from object-specific behavior.

The player only needs to determine which interactable object is being targeted and send an interaction request. The individual object decides what that interaction actually does.

This structure is intended to support systems such as:

- Doors
- Keys and item pickups
- Puzzle objects
- Environmental distractions
- Notes and audio logs
- Other contextual interactions

The complete development repository is private. Selected excerpts are shown here for portfolio purposes.
