# FLM Page - screen properties

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
ClearCollect(
    colSSPRaw,
    AddColumns(
        Filter('Attach Attack', 'Upsell Eligible' <> "No"),
        'SSPRegion', LookUp(PowerBIIntegration.Data, 'HPE Opportunity Id' = 'Opp ID').'SSP Region',
        'ServicesOSVal', IfError(Value(LookUp(PowerBIIntegration.Data, 'HPE Opportunity Id' = 'Opp ID').'Services OS'), 0)
    )
);

ClearCollect(
    colSSPChart,
    ForAll(
        GroupBy(
            Filter(colSSPRaw, !IsBlank(SSPRegion) && SSPRegion <> ""),
            'SSPRegion', 'RegionRows'
        ),
        {
            Region: SSPRegion,
            TotalOS: Sum(RegionRows, ServicesOSVal)
        }
    )
);
With(
    {
        maxVal: Max(colSSPChart, TotalOS),
        barCount: CountRows(colSSPChart),
        indexed: ForAll(
            Sequence(CountRows(colSSPChart)),
            With(
                {row: Index(colSSPChart, Value)},
                {
                    Region: row.Region,
                    TotalOS: row.TotalOS,
                    idx: Value - 1
                }
            )
        )
    },
    Set(varSSPMaxVal, maxVal);
    Set(varSSPBarCount, barCount);
    Set(
    varSSPBars,
    Concat(
        indexed,
        With(
            {
                barH: If(maxVal > 0, Round(ThisRecord.TotalOS / maxVal * 125, 0), 0),
                xPos: 52 + ThisRecord.idx * Round(273 / barCount, 0) + Round(Round(273 / barCount, 0) * 0.15, 0),
                barW: Round(Round(273 / barCount, 0) * 0.5, 0),
                yBar: 195 - If(maxVal > 0, Round(ThisRecord.TotalOS / maxVal * 125, 0), 0),
                del: Text(0.5 + ThisRecord.idx * 0.1, "[$-en-US]0.00"),
                cleanRegion: Substitute(Substitute(Substitute(Substitute(
                    ThisRecord.Region, "&", "&amp;"), "'", " "), "<", "&lt;"), ">", "&gt;"),
                valLabel: If(ThisRecord.TotalOS >= 1000000,
                    Text(ThisRecord.TotalOS / 1000000, "[$-en-US]0.0") & "M",
                    If(ThisRecord.TotalOS >= 1000,
                        Text(ThisRecord.TotalOS / 1000, "[$-en-US]0") & "K",
                        Text(ThisRecord.TotalOS, "[$-en-US]0")
                    )
                )
            },
            "<rect x='" & Text(xPos) & "' y='" & Text(yBar) & "' width='" & Text(barW) & "' height='" & Text(barH) & "' rx='3' ry='3' fill='#04909d' opacity='0.9'>" &
            "<animate attributeName='height' from='0' to='" & Text(barH) & "' dur='0.6s' begin='" & del & "s' fill='freeze' calcMode='spline' keySplines='0.16 1 0.3 1'/>" &
            "<animate attributeName='y' from='195' to='" & Text(yBar) & "' dur='0.6s' begin='" & del & "s' fill='freeze' calcMode='spline' keySplines='0.16 1 0.3 1'/>" &
            "</rect>" &
            "<text x='" & Text(xPos + Round(barW / 2, 0)) & "' y='" & Text(yBar - 4) & "' font-family='Arial, Helvetica, sans-serif' font-size='8' fill='#006750' font-weight='700' text-anchor='middle'>" & valLabel & "</text>" &
            "<text x='" & Text(xPos + Round(barW / 2, 0)) & "' y='210' font-family='Arial, Helvetica, sans-serif' font-size='8' fill='#606a70' text-anchor='middle'>" & If(Len(cleanRegion) > 9, Left(cleanRegion, 9) & "..", cleanRegion) & "</text>"
        ),
        ""
    )
));
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
```
