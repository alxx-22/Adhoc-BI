# Summer Score — screen properties

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
Set(varAnimKey1, If(IsBlank(varAnimKey1), 1, varAnimKey1 + 1));ClearCollect(
    colEngagementMetrics,
    ForAll(
        Split(First(PowerBIIntegration.Data).PackedEngagement, "||"),
        {
            UniqueUser:                     IfError(Index(Split(Value, "//"), 1).Value, ""),
            WeeklyScore:                    IfError(Index(Split(Value, "//"), 2).Value, ""),
            IBNSPoints:                     IfError(Index(Split(Value, "//"), 3).Value, ""),
            BookingRate:                    IfError(Index(Split(Value, "//"), 4).Value, ""),
            AttachUpsellPoints:             IfError(Index(Split(Value, "//"), 5).Value, ""),
            BacklogPoints:                  IfError(Index(Split(Value, "//"), 6).Value, ""),
            Manager:                        IfError(Index(Split(Value, "//"), 7).Value, ""),
            LeadingEdgeCompletionPoints:    IfError(Index(Split(Value, "//"), 8).Value, ""),
            DisplayName:                    IfError(Index(Split(Value, "//"), 9).Value, ""),
            Country:                        IfError(Index(Split(Value, "//"), 10).Value, "")
        }
    )
);

ClearCollect(
    colSorted,
    SortByColumns(
        AddColumns(colEngagementMetrics,
            'TotalPts',
            Value(IBNSPoints) + Value(AttachUpsellPoints) + Value(BacklogPoints) + Value(LeadingEdgeCompletionPoints)
        ),
        "TotalPts",    SortOrder.Descending,
        "WeeklyScore", SortOrder.Descending
    )
);

ClearCollect(
    colEngagementRanked,
    ForAll(
        Sequence(CountRows(colSorted)),
        {
            Rank:                        Value,
            UniqueUser:                  Index(colSorted, Value).UniqueUser,
            WeeklyScore:                 Index(colSorted, Value).WeeklyScore,
            IBNSPoints:                  Index(colSorted, Value).IBNSPoints,
            BookingRate:                 Index(colSorted, Value).BookingRate,
            AttachUpsellPoints:          Index(colSorted, Value).AttachUpsellPoints,
            BacklogPoints:               Index(colSorted, Value).BacklogPoints,
            Manager:                     Index(colSorted, Value).Manager,
            LeadingEdgeCompletionPoints: Index(colSorted, Value).LeadingEdgeCompletionPoints,
            DisplayName:                 Index(colSorted, Value).DisplayName,
            Country:                     Index(colSorted, Value).Country
        }
    )
);
```
