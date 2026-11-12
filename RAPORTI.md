Raporti i studentit: UrbanBike AI

Studenti: Elvin Mefaja,Ira Murataj,Ardion Jaupaj,Alesio Osmanaj 
Data: 2026-11-11 
GitHub-i im: https://github.com/Elvi111/urbanbike-ai-student
Figma: (https://www.figma.com/design/yWzCQ2wy13GsEmeCU6yg0e/Untitled?node-id=0-1&m=dev&t=BVPLNpi2d8t6g1i7-1)

Qëllimi

Problemi që zgjidhim: sa qira biçikletash priten për ditën pasuese në Washington, D.C.?

Target-i y: rentals (numri total i qirave ditore).

Predictorët X: qiratë e një dite më parë (rentals_lag1), të 7 ditëve më parë (rentals_lag7) dhe mesatarja e 7 ditëve të kaluara (rentals_mean7); temperatura, reshjet dhe era me vonesë 7 ditore (temp_c_lag7, rain_mm_lag7, wind_ms_lag7); trend_days (numri i ditëve nga fillimi); dhe kategoritë weekday, month, workingday, holiday.

Koha kur parashikojmë dhe çfarë informacioni është i njohur: parashikojmë një ditë përpara. Supozohet se numërimet e qirave të ditës së djeshme dhe të ditëve më të hershme janë të njohura. Moti përdoret me vonesë 7 ditore. Të dhënat janë historike, 2011–2012.

Pastrimi

Burimet reale dhe çelësi i bashkimit: UCI Bike Sharing (day.csv, 731 ditë, qiratë) dhe NASA POWER (nasa_weather.csv, temperatura, reshjet dhe era për Washington D.C.). Bashkohen sipas datës (date), me kontroll një-me-një.

Dallimi mes origjinaleve dhe kopjes didaktike: dy burimet reale nuk ndryshohen (UCI ka 0 mungesa). Gabimet krijohen me qëllim vetëm te kopja data/teaching/dirty.csv, që pastrimi të demonstrohet.

Rreshtat para/pas: kopja didaktike ka 743 rreshta. Pas heqjes së dublimeve mbeten 731. Pas ndërtimit të lag-eve (7 ditët e para humbasin) mbeten 724 rreshta për modelim.

Dublimet: 12 rreshta identikë, të hequr me drop_duplicates().

Mungesat para/pas konvertimit: para konvertimit kishte 30 qeliza boshe (18 te temp_c, 12 te rain_mm). Gjashtë vlera në temp_c ishin teksti "gabim", që pd.to_numeric(errors="coerce") e kthen në NaN. Pas konvertimit janë 36 mungesa meteorologjike. Tipet u rregulluan: date nga object në datetime64[ns], temp_c nga object në float64.

Arsyeja e imputimit vetëm me train: mungesat plotësohen me medianën brenda Pipeline, e mësuar vetëm nga periudha train. Nëse mediana llogaritej me të gjitha të dhënat, informacion nga validation dhe test (e ardhmja) do të futej në trajnim dhe rezultati do të dukej më i mirë se ç'është në të vërtetë. Mungesa nuk trajtohet si temperaturë 0°C.

Grafikët

Vëzhgimi 1 dhe figura përkatëse (01_demand.png): qiratë ditore kanë sezonalitet të qartë. Janë rreth 1000–2000 në janar 2011, rriten në rreth 5000 nga maji deri në shtator 2011, bien në dimër dhe rriten përsëri në pranverën 2012 (deri në 6000–8000). Mesatarja në train është 3446 qira në 2011 dhe 4454 në 2012, pra ka edhe rritje nga viti në vit. Seria ditore luhatet shumë, ndërsa mesatarja 7-ditore tregon prirjen.

Vëzhgimi 2 dhe figura përkatëse (03_weather.png): kur temperatura (7 ditë më parë) rritet nga rreth −5°C në rreth 15°C, qiratë tentojnë të rriten, por mbi rreth 20°C kurba rrafshohet rreth 4000–5000. Pikat janë shumë të shpërndara (korrelacioni linear është rreth 0.54), kështu që temperatura e vetme nuk e parashikon saktë kërkesën. Në 02_workingday.png ditët e punës kanë mesatare 3761 qira kundrejt 3570 për fundjavat dhe festat, pra një diferencë e vogël. Mesatarja nuk tregon gjithë shpërndarjen.

Pse një lidhje e dukshme nuk provon shkakësi: temperatura dhe qiratë mund të rriten bashkë sepse të dyja ndjekin stinën, ose sepse ndikojnë faktorë të tjerë (rritja e shërbimit nga 2011 në 2012, festat, reshjet). Scatter plot-i tregon vetëm bashkëlidhje, jo se njëra shkakton tjetrën.

Vlerësimi

Ndarja kohore: train 2011-01-08 deri 2012-04-30 (479 rreshta), validation 2012-05-01 deri 2012-08-31 (123 rreshta), test 2012-09-01 deri 2012-12-31 (122 rreshta). Train mëson parametrat dhe imputimin, validation zgjedh modelin dhe testi përdoret në fund.

Modeli	Validation MAE	Test MAE	Test RMSE	Test R²
Yesterday	775.203	916.402	1330.386	0.539
DummyMean	2953.866	2494.628	2790.064	-1.027
Ridge	687.558	893.251	1252.655	0.591
RandomForest	1821.528	1671.499	2042.654	-0.087

Modeli i zgjedhur nga validation dhe arsyeja: Ridge, sepse ka MAE më të ulët në validation (687.558) mes dy kandidatëve ML (Ridge dhe Random Forest).

Rezultati i provës me Random Forest: del më keq se Ridge dhe më keq se baseline-i "Yesterday" (test MAE 1671.499, R² negativ −0.087). Një model më kompleks nuk është automatikisht më i mirë. Trajnimi ka vetëm 479 rreshta dhe kërkesa në testin e vjeshtës 2012 është mesatarisht rreth 5687 qira, më e lartë se mesatarja e train (rreth 3701). Pemët nuk nxjerrin vlera mbi ato që kanë parë në train, ndërsa Ridge, që është linear, i ndjek më mirë nivelet e reja.

Interpretimi i gabimit në qira për ditë: Ridge gabon mesatarisht 893 qira në ditë (MAE) në test, rreth 16% të mesatares së testit. Përmirësimi ndaj baseline-it "Yesterday" (916 qira) është i vogël. RMSE 1253 është më i lartë se MAE sepse penalizon gabimet e mëdha.

Rezultati i klasifikimit, nëse u realizua: "kërkesë e lartë" përcaktohet si rentals ≥ medianës së train (3907 qira). Në test ka 103 ditë të larta dhe 19 normale.

Modeli	Accuracy	Balanced accuracy	Macro-F1
DummyMostFrequent	84.43%	50.00%	0.458
LogisticRegression	91.80%	77.98%	0.820

Matrica e konfuzionit e LogisticRegression është [[11, 8], [2, 101]]: 11 ditë normale u njohën saktë, 8 normale u parashikuan si të larta, 2 të larta u parashikuan si normale dhe 101 të larta u njohën saktë. Baseline-i arrin accuracy 84.43% duke thënë gjithmonë "e lartë", por balanced accuracy 50% tregon se nuk dallon asnjë ditë normale. Prandaj, kur klasat janë të pabalancuara, accuracy duhet lexuar bashkë me balanced accuracy dhe macro-F1.

Kufizimet
Të dhënat janë nga një qytet (Washington, D.C.) dhe një periudhë historike (2011–2012). Rezultatet nuk garantojnë se do të vlejnë për qytete ose vite të tjera.
Të dhënat e motit nga NASA vijnë nga një rrjet meteorologjik (me standard kohor LST), jo nga një stacion lokal. Përdorimi i vonesës 7-ditore nuk i zgjidh të gjitha kufizimet e reanalizës historike.
Parashikimi njëditor supozon se qiratë e ditës së djeshme janë të njohura. Aplikacioni është demonstrim historik mbi datat e testit, jo shërbim i testuar për kërkesën e sotme.
Kontributet individuale
Studenti	Skedari/ekrani	Puna dhe vendimi	Commit/prova
EMRI MBIEMRI	config/common.py, 01_clean.py	Ndërtova pastrimin dhe ndarjet kohore; ekzekutova dhe verifikova numrat (731 rreshta, 12 dublime, 36 mungesa)	reports/data_quality.json
EMRI MBIEMRI	02_visualize.py, 03_train.py, 04_classify.py	Krijova grafikët dhe modelet, zgjodha Ridge sipas validation MAE dhe interpretova rezultatet	reports/regression_metrics.csv, reports/classification_metrics.json
EMRI MBIEMRI	app.py, run_all.py, Figma	Nisa aplikacionin lokal dhe e provova për 2012-09-01 (parashikim 6834.73, reale 6140); skicova konceptin në Figma	Commit-i dhe GitHub-i im
Burimet

UCI: https://doi.org/10.24432/C5W894 NASA: https://power.larc.nasa.gov/docs/services/api/temporal/daily/ Provenienca lokale: data/raw/provenance.json