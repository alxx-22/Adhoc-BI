# FLM Page — screen properties

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
ClearCollect(
    colAAOppIds,
    'Attach Attack'
);

ClearCollect(
    colRepCompletion,
    ForAll(
        GroupBy(
            PowerBIIntegration.Data,
            'Feedback Progress',
            'OppRows'
        ),
        {
            Label: 'Feedback Progress',
            TotalOpps: CountRows(OppRows),
            Completed: CountRows(
                Filter(
                    OppRows,
                    'HPE Opportunity Id' in colAAOppIds.'Opp ID'
                )
            ),
            PctComplete: If(
                CountRows(OppRows) > 0,
                Round(
                    CountRows(Filter(
                        OppRows,
                        'HPE Opportunity Id' in colAAOppIds.'Opp ID'
                    )) / CountRows(OppRows) * 100,
                    0
                ),
                0
            )
        }
    )
);
ClearCollect(colHasRecord,
    Filter('Attach Attack', !IsBlank('Opp ID'))
);
Set(varSlicerToggle, 0);
Set(varSlicerOpen, false);Set(varFCSTToggle, 0);
Set(varFCSTOpen, false);
Set(
    varUntrackedOS,
    Sum(
        Filter(
            PowerBIIntegration.Data,
            IsBlank(LookUp(colHasRecord, 'Opp ID' = 'HPE Opportunity Id'))
        ),
        IfError(Value('Services OS'), 0)
    )
);
Set(
    varUntrackedCount,
    CountRows(
        Filter(
            PowerBIIntegration.Data,
            IsBlank(LookUp(colHasRecord, 'Opp ID' = 'HPE Opportunity Id'))
        )
    )
);
Set(
    varTrackedOS,
    Sum(
        Filter(
            PowerBIIntegration.Data,
            !IsBlank(LookUp(
                Filter('Attach Attack', 'Upsell Eligible' <> "No"),
                'Opp ID' = 'HPE Opportunity Id'
            ))
        ),
        IfError(Value('Services OS'), 0)
    )
);
Set(
    varTrackedCount,
    CountRows(Filter('Attach Attack', 'Upsell Eligible' <> "No"))
);
ClearCollect(
    colOSDelta,
    ForAll(
        Filter('Attach Attack', 'Upsell Eligible' <> "No"),
        With(
            {
                currentOS: IfError(Value(LookUp(PowerBIIntegration.Data, 'HPE Opportunity Id' = 'Opp ID').'Services OS'), 0),
                trackedOS: IfError(Value(ThisRecord.'Services OS'), 0)
            },
            {
                OppId: 'Opp ID',
                Delta: currentOS - trackedOS
            }
        )
    )
);

Set(varOSDeltaSum, Sum(Filter(colOSDelta, Delta > 0), Delta));
Set(varOSDeltaCount, CountRows(Filter(colOSDelta, Delta > 0)))


// ---- Rep coverage & gaps: grouped live from PowerBIIntegration.Data (same principle as the
//      app's colRepCompletion), so a report filter on [Entitled Manager Name] narrows it. ----
ClearCollect(
    colFPDetail,
    ForAll(
        GroupBy(PowerBIIntegration.Data, 'Feedback Progress', 'OppRows'),
        With(
            {
                trk: Filter(OppRows, 'HPE Opportunity Id' in colAAOppIds.'Opp ID'),
                unt: Filter(OppRows, !('HPE Opportunity Id' in colAAOppIds.'Opp ID'))
            },
            {
                Rep:         'Feedback Progress',
                TrackedOS:   IfError(Sum(trk, IfError(Value('Services OS'), 0)), 0),
                UntrackedOS: IfError(Sum(unt, IfError(Value('Services OS'), 0)), 0),
                CC:          CountRows(Filter(unt, 'Target Opp?' = "CC / Day 1 Upsell")),
                LP:          CountRows(Filter(unt, 'Target Opp?' = "Low Pen Rate")),
                NS:          CountRows(Filter(unt, 'Target Opp?' = "No Services Op"))
            }
        )
    )
);
ClearCollect(colFPDetailR, Filter(colFPDetail, Rep <> "" && (TrackedOS + UntrackedOS) > 0));
Set(varFPMax, Max(colFPDetailR, TrackedOS + UntrackedOS));
```
