# Homepage - screen properties

Select the screen in the tree view, pick each property in the dropdown,
and paste the formula (everything after the `=`) into the formula bar.

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
Reset(Timer2);
Set(varAnimKey1, If(IsBlank(varAnimKey1), 1, varAnimKey1 + 1));
```
