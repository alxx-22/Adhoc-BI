# Loading — screen properties

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
Set(varEntered, false);
Set(varAnimKey1, If(IsBlank(varAnimKey1), 1, varAnimKey1 + 1));
```
