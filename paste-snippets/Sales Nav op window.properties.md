# Sales Nav op window — screen properties

Paste each formula (after the `=`) into the formula bar.

## Fill

```
RGBA(247, 247, 247, 1)
```

## LoadingSpinnerColor

```
RGBA(1, 169, 130, 1)
```

## OnHidden

```
Set(timerotoole,false);
Set(timerdog,true)
```

## OnVisible

```
Set(Container2Visible, false);
Set(Container1Visible,false);
Set(Container3Visible,false);
Set(varAnimKey, 0);
Set(varAnimKey1, If(IsBlank(varAnimKey1), 1, varAnimKey1 + 1));
Set(varAnimKey3, 0);
UpdateContext({
    varShowAltOpp: With(
        {rec: LookUp('Attach Attack', 'Opp ID' = thisop)},
        If(IsBlank(rec.'No Upsell Reason'), "", rec.'No Upsell Reason')
    ) = "Day1/9X quoted on seperate op."
});
UpdateContext({varReasonChanged: false});
Set(varProdToggle, 0);
Set(varOSToggle, 0);
Set(varNonOSToggle, 0);
Set(timerotoole,true);
```
