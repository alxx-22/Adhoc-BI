# Sales Nav List — screen properties

Paste each formula (after the `=`) into the formula bar.

## Fill

```
RGBA(247, 247, 247, 1)
```

## LoadingSpinnerColor

```
RGBA(1, 169, 130, 1)
```

## OnVisible

```
Set(varAnimKey1, If(IsBlank(varAnimKey1), 1, varAnimKey1 + 1));
Set(varAnimKey3, 0);
Set(notibox,false);
Set(showCC,false);
Set(varCCtoggle,0);
Set(showNSop,false);
Set(varNSoptoggle,0);
Set(varLPRtoggle,0);
Set(showLPR,false);
Set(varNEWToggle,0);
Set(showNEW,false);
Set(vartrackToggle,0);
Set(showtracking,false);
ClearCollect(colTrackedByMe, 
    Filter('Attach Attack', 'Modified By'.Email = User().Email)
);
ClearCollect(colHasRecord,
    Filter('Attach Attack', !IsBlank('Opp ID'))
);
```
