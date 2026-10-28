# Värderingsmotor

Ett API som värderar börsnoterade bolag med flera metoder och väger ihop dem till ett modellvärde. Byggt i Python med FastAPI och driftsatt som webbtjänst. Detta repo innehåller backend; gränssnittet anropar endpoints nedan.

## Vad den gör

För en ticker (t.ex. `AAPL` eller `VOLV-B.ST`) hämtas kurs från Twelve Data och fundamentaldata från Financial Modeling Prep. Bolaget värderas sedan med:

| Metod | Vikt | Kort beskrivning |
| --- | --- | --- |
| Diskonterade vinster per aktie | 35 % | Vinst per aktie i tre faser (5 + 5 år + evig tillväxt), diskonterad med avkastningskravet på eget kapital |
| P/E mot sektor | 20 % | Vinst per aktie gånger en sektormultipel, justerad för ROE |
| EV/EBITDA mot sektor | 20 % | EBITDA gånger en sektormultipel, justerad för tillväxt och marginal, omräknat till värde per aktie |
| Diskonterat fritt kassaflöde | 15 % | Fritt kassaflöde per aktie, samma struktur som ovan |
| Relativvärdering mot jämförelsebolag | 10 % | Median-P/E och EV/EBITDA för fyra jämförelsebolag (valfri, kräver AI-nyckel) |
| P/B och Grahams formel | 0 till 10 % | Referensmodeller, ingår bara där de är rimliga (t.ex. inte för tech) |

Resultatet anges som avstånd till modellvärdet: under, nära eller över.

## Antaganden

- Avkastningskrav på eget kapital: 4 % riskfri ränta + beta × 5,5 % riskpremie (CAPM), begränsat till 7 till 15 %.
- Evig tillväxt 2,5 %. Tillväxt år 1 till 5 bygger på senaste omsättningstillväxten, begränsad till rimliga nivåer. Saknas data används 5 % och svaret flaggar det (`growth_assumed`).
- Sektormultiplarna är egna schabloner och används som fallback. Relativvärdering mot faktiska jämförelsebolag är att föredra.

## Begränsningar

- Vinst per aktie används som ersättning för kassaflöde till aktieägarna, en förenkling.
- Modellen använder senaste årets siffror och tar inte hänsyn till konjunktur, engångsposter eller kapitalstruktur över tid.
- Datakvaliteten beror på API-leverantörerna; koden innehåller rimlighetskontroller för fritt kassaflöde.
- Modellvärden är inte investeringsrådgivning.

## Köra lokalt

```
pip install -r requirements.txt
export FMP_API_KEY=din_nyckel
export TWELVE_DATA_KEY=din_nyckel
export ANTHROPIC_API_KEY=din_nyckel   # valfri, för jämförelsebolag
uvicorn main:app --reload
```

Endpoints: `/value/{ticker}`, `/value-with-peers/{ticker}`, `/compare?tickers=AAPL,MSFT`.
