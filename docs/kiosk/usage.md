# Kiosk — Usage

## 1. Installation

Follow the [Installation](../getting-started/installation.md) guide and add `"Kiosk"` to your `Build.cs`.

## 2. Create a Theme Data Asset

1. Right-click → **Data Asset** → `UKioskThemeData`
2. Configure colors, fonts, and spacing to match your game's visual identity

## 3. Apply the Theme

```cpp
UKioskSubsystem* Kiosk = GetGameInstance()->GetSubsystem<UKioskSubsystem>();
Kiosk->SetTheme(MyThemeDataAsset);
```

## 4. Create a Screen

Subclass `SKioskScreen` in C++:

```cpp
class SMyInventoryScreen : public SKioskScreen
{
public:
    SLATE_BEGIN_ARGS(SMyInventoryScreen) {}
    SLATE_END_ARGS()

    void Construct(const FArguments& InArgs);

    virtual TSharedRef<SWidget> BuildContent() override
    {
        return SNew(SVerticalBox)
            + SVerticalBox::Slot()
            [
                SNew(SKioskButton)
                    .Label(LOCTEXT("Close", "Close"))
                    .OnClicked(this, &SMyInventoryScreen::OnCloseClicked)
            ];
    }

    FReply OnCloseClicked()
    {
        UKioskSubsystem::GetSubsystem()->Pop();
        return FReply::Handled();
    }
};
```

## 5. Push a Screen

```cpp
Kiosk->Push(SNew(SMyInventoryScreen));
```

## 6. Pop a Screen

```cpp
Kiosk->Pop();
```

## 7. Show a Modal Dialog

```cpp
Kiosk->ShowModal(
    LOCTEXT("Confirm", "Are you sure?"),
    LOCTEXT("ConfirmBody", "This action cannot be undone."),
    FSimpleDelegate::CreateUObject(this, &AMyGameMode::OnConfirm),
    FSimpleDelegate::CreateUObject(this, &AMyGameMode::OnCancel)
);
```

## 8. Show a Toast Notification

```cpp
Kiosk->ShowToast(
    LOCTEXT("Saved", "Game Saved"),
    3.0f  // auto-dismiss after 3 seconds
);
```

## 9. Switch Theme at Runtime

```cpp
// Color-blind mode
Kiosk->SetTheme(ColorBlindThemeAsset);
```

All screens instantly re-render using the new theme.

## 10. Blueprint Usage

All Kiosk operations are Blueprint-callable. Screen creation from Blueprint requires implementing `UKioskScreenBase` (the Blueprint-accessible wrapper around `SKioskScreen`).
