# CSA313 лаборатори №6: Category-Partition ба PICT

**Оюутны нэр:** Эрдэнэбаярын Энхсаруул

**Оюутны код:** B232270145

**Хичээл:** Программ хангамжийн чанарын баталгаа ба тест (F.CSA313)

## A. Category-Partition арга

**Тестлэх функц:** `POST /transfer { "from": ID, "to": ID, "amount": NUM, "currency": CODE }`

### A1. Сонголт ба төлөөлөх утгууд

| Параметр | Сонголт | Төлөөлөх утгууд |
|---|---|---|
| `from` | Илгээгч дансны төлөв | Valid, Frozen, Missing |
| `to` | Хүлээн авагчийн төлөв, `from`-той харьцаа | Valid, Missing, SameAsFrom |
| `amount` | Дүнгийн утга, хэмжээ | Zero, Normal, OverBalance, OverLimit |
| `currency` | Дансны валюттай харьцуулалт | Same, Different |

**Далд орчны сонголт (setup):**

- Өдрийн лимит: Normal / Exceeded (`OverLimit` үүсгэнэ).
- Үлдэгдэл: Sufficient / Insufficient (`OverBalance` үүсгэнэ).
- Ханш: 1 / өөр ханш (`Different` үед хөрвүүлэлт хийнэ).

### A2. Тэмдэглэгээ, хязгаарлалт ба комбинацийн тооцоо

- `[SINGLE]` нь `from = Missing` эсвэл `to = Missing` үед бусад утгаас үл хамааран тухайн Missing нөхцөлийг нэг representative кейсээр шалгахыг заана.
- `[ERROR]` нь `Frozen`, `SameAsFrom`, `Zero`, `OverBalance`, `OverLimit` утга тус бүрийг өөрийн алдааны ангитай нь шалгахыг заана.
- `transfer.model` загварт `from` эсвэл `to` нь Missing бол `amount` Normal байх `IF` хязгаарлалт тавьсан. Энэ нь шилжүүлэг хийх боломжгүй данстай үед дүнгийн алдааг давхар шалгахаас сэргийлнэ.
- **`~` хэрэглэхэд энэ `IF` хязгаарлалт хэрэгтэй хэвээр юу? — Үгүй, илүүц болно.** PICT нь `~`-тэй (сөрөг) утгыг нэг мөрөнд зөвхөн нэгийг оруулж, бусад параметрт заавал эерэг утга сонгодог. `transfer-neg.model`-д `Missing` нь `~Missing`, Amount-ын эерэг утга нь зөвхөн `Normal` тул `From = ~Missing` эсвэл `To = ~Missing` мөрөнд Amount автоматаар `Normal` болно — `IF`-ийн шаардах нөхцөл аль хэдийн биелсэн байна. Мөн `~Frozen + ~Zero` гэх мэт бусад давхар алдааг ч `~` өөрөө хориглодог тул `IF` нэмэлт хамгаалалт өгөхгүй.
  - **Туршилтаар баталгаажуулсан:** `transfer-neg.model`-оос хоёр `IF` мөрийг хасаад PICT ажиллуулахад гаралт [results/transfer-neg-noif.txt](./results/transfer-neg-noif.txt) нь [results/transfer-neg.txt](./results/transfer-neg.txt)-тэй **мөр бүрээрээ ижил (16 мөр)** гарсан (`diff` ялгаагүй).
    ```bash
    grep -v '^IF' transfer-neg.model > /tmp/neg-noif.model
    ./tools/pict/pict /tmp/neg-noif.model > results/transfer-neg-noif.txt
    diff results/transfer-neg-noif.txt results/transfer-neg.txt   # ялгаа байхгүй
    ```
  - Иймээс `IF` хязгаарлалт зөвхөн `~`-гүй `transfer.model`-д утга учиртай; `transfer-neg.model`-д уншигчид зорилгыг ойлгуулах тайлбар төдий үүрэгтэй, үр дүнд нөлөөлөхгүй.
- Бүрэн Cartesian олонлог: `N = 3 × 3 × 4 × 2 = 72`.
- Хязгаарласан representative олонлог: хоёр амжилттай валютын хувилбар + долоон алдааны анги (Missing from, Frozen, Missing to, SameAsFrom, Zero, OverBalance, OverLimit) = `M = 9`. Ингэснээр ангиллын үндсэн тохиолдол `72 → 9` болно. A3-ын 10 дахь мөр нь эдгээр есөн ангид нэмэх, давхар алдааны давуу эрэмбийг шалгах тусгай тохиолдол юм.
- Давхар алдааны шалгах эрэмбэ: `from → to → amount → currency`. Иймээс `from` дээрх алдаа бусад алдаанаас түрүүлж буцаагдана.

### A3. Transfer функцийн спецификацийн хүснэгт

| № | From | To | Amount | Currency | Хүлээгдэх үр дүн (Oracle) | Тайлбар |
|---:|---|---|---|---|---|---|
| 1 | Valid | Valid | Normal | Same | `201 OK` | Амжилттай шилжүүлэг |
| 2 | Valid | Valid | Normal | Different | `201 OK` | Валют хөрвүүлэлт |
| 3 | Missing | Valid | Normal | Same | `200 ERROR_NO_FROM` | `[SINGLE]` |
| 4 | Frozen | Valid | Normal | Same | `200 ERROR_FROZEN` | Frozen дансны алдаа |
| 5 | Valid | Missing | Normal | Same | `200 ERROR_NO_TO` | `[SINGLE]` |
| 6 | Valid | SameAsFrom | Normal | Same | `200 ERROR_SAME_ACCOUNT` | Ижил дансны алдаа |
| 7 | Valid | Valid | Zero | Same | `200 ERROR_BAD_AMOUNT` | Дүн тэг |
| 8 | Valid | Valid | OverBalance | Same | `200 ERROR_INSUFFICIENT` | Үлдэгдэл хүрэлцэхгүй |
| 9 | Valid | Valid | OverLimit | Same | `200 ERROR_LIMIT` | Өдрийн лимит хэтэрсэн |
| 10 | Frozen | Missing | OverBalance | Different | `200 ERROR_FROZEN` | Давхар алдаа; `from` түрүүлнэ |

## B. Microsoft PICT

PICT эх кодыг репод оруулаагүй. Build-д ашигласан Microsoft PICT эхийн commit нь `ab76c2548f551fcb46314e58653a1bd1172f3a72`; эх кодоос бүтээж ажиллуулсан командын бүртгэл [results/pict-build.txt](./results/pict-build.txt)-д байна.

### B1. Model-уудын мөрийн тооны харьцуулалт

Доорх тоо нь үр дүнгийн файлын толгой мөрийг хассан тестийн мөрийн тоо юм.

| Model | Хамралт / нөхцөл | Тестийн мөр |
|---|---|---:|
| `browser.model` | Pairwise (2-way) | 10 |
| `browser.model` | 3-way | 20 |
| `transfer-nc.model` | Хязгаарлалтгүй pairwise | 13 |
| `transfer.model` | Missing үед Normal гэсэн `IF` хязгаарлалттай | 12 |
| `transfer-neg.model` | `~` сөрөг утгын тэмдэглэгээтэй | 16 |

Үр дүнгүүд: [browser-2way](./results/browser-2way.txt), [browser-3way](./results/browser-3way.txt), [transfer-nc](./results/transfer-nc.txt), [transfer](./results/transfer.txt), [transfer-neg](./results/transfer-neg.txt).

### B1.1. Transfer-ийн гурван хувилбарын харьцуулалт: happy path ба нэг мөр дэх алдааны тоо

**Тоолох дүрэм:** Алдааны утга гэж `From ∈ {Frozen, Missing}`, `To ∈ {Missing, SameAsFrom}`, `Amount ∈ {Zero, OverBalance, OverLimit}`-ийг хэлнэ. `Currency` (Same/Different) хоёулаа хүчинтэй тул алдаа биш. **Happy path** = `Valid, Valid, Normal` ба дурын валют (алдааны тоо 0). Мөр бүрийн алдааны тоог `results/*.txt` файлаас шууд тоолсон.

| Model | Нийт мөр | Happy path (0 алдаа) | 1 алдаатай | 2 алдаатай | 3 алдаатай |
|---|---:|---:|---:|---:|---:|
| `transfer-nc` (хязгаарлалтгүй) | 13 | **1** (мөр 1) | 1 (мөр 10) | 8 | 3 (мөр 3, 6, 9) |
| `transfer` (`IF` хязгаарлалттай) | 12 | **0** | 3 (мөр 2, 3, 6) | 7 | 2 (мөр 1, 10) |
| `transfer-neg` (`~` тэмдэглэгээтэй) | 16 | **2** (мөр 1, 2) | 14 (мөр 3–16) | 0 | 0 |

**Мөр бүр бодит байдалд аль алдааг буцаах вэ** (шалгах эрэмбэ `from → to → amount`, A2-ыг үз):

| Model | `OK` | `NO_FROM` | `FROZEN` | `NO_TO` | `SAME_ACCOUNT` | `BAD_AMOUNT` | `INSUFFICIENT` | `LIMIT` |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `transfer-nc` | 1 | 4 | 4 | 2 | 2 | **0** | **0** | **0** |
| `transfer` | **0** | 3 | 5 | 1 | 2 | **0** | 1 | **0** |
| `transfer-neg` | 2 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |

**Дүгнэлт:**

- **Мөрийн тоо цөөн байх нь сайн гэсэн үг биш.** `transfer-nc` (13) ба `transfer` (12) нь цөөн мөртэй боловч ихэнх мөрөнд 2–3 алдаа зэрэг орсон. Систем эхний алдаан дээр буцдаг тул (error masking) үлдсэн алдаанууд хэзээ ч шалгагдахгүй. Жишээ нь `transfer-nc`-ийн мөр 3 (`Missing, SameAsFrom, Zero`) зөвхөн `ERROR_NO_FROM`-ийг шалгана; `SameAsFrom`, `Zero` далдлагдана.
- **`transfer-nc`:** happy path ердөө 1, нэг алдаатай мөр 1. `Zero`, `OverBalance`, `OverLimit` алдаа гаралтад орсон ч бүгд өөр алдаатай хамт байгаа тул `BAD_AMOUNT`, `INSUFFICIENT`, `LIMIT` гэсэн гурван oracle нэг ч удаа ажиллахгүй.
- **`transfer`:** `IF` хязгаарлалт нь зөвхөн Missing-тэй мөрийн Amount-ыг Normal болгосон тул нэг алдаатай мөр 1 → 3 болсон, гэвч **happy path 0** болж амжилттай шилжүүлэг огт тестлэгдэхгүй. `BAD_AMOUNT`, `LIMIT` мөн далдлагдсан.
- **`transfer-neg`:** PICT `~` утгыг нэг мөрөнд хоёроос олныг хослуулдаггүй тул мөр бүр 0 эсвэл 1 алдаатай. Мөрийн тоо хамгийн их (16) ч happy path 2 (MNT ба хөрвүүлэлттэй), алдааны 7 утга тус бүр яг 2 удаа (Same/Different валюттай) тусдаа шалгагдаж, бүх 8 oracle хамрагдсан. Иймээс сөрөг тест хийхэд хамгийн тохиромжтой хувилбар нь `transfer-neg`.

### B2. PICT гаралтаас гаргасан бодит тест кейсүүд

Доорх `transfer-neg` мөрийн дугаарууд нь толгой мөрийг тооцоогүй PICT-ийн өгөгдлийн мөрийн дугаар юм.

1. **Амжилттай шилжүүлэг — `transfer-neg`, мөр 1 (`Valid, Valid, Normal, Same`).**
   - **Setup:** `A1` идэвхтэй, үлдэгдэл 500,000₮, өдрийн лимит 1,000,000₮; `B1` идэвхтэй; валют MNT.
   - **Оролт:** `POST /transfer {"from":"A1","to":"B1","amount":50000,"currency":"MNT"}`
   - **Oracle:** `201 OK`, `{"result":"OK"}`.
2. **Нэг алдаатай кейс — `transfer-neg`, мөр 8 (`~Missing, Valid, Normal, Same`).**
   - **Setup:** `INVALID_ACC` данс байхгүй; `B1` идэвхтэй; дүн 50,000₮-ийг шилжүүлэхэд хангалттай үлдэгдэл, лимиттэй.
   - **Оролт:** `POST /transfer {"from":"INVALID_ACC","to":"B1","amount":50000,"currency":"MNT"}`
   - **Oracle:** `200 OK`, `{"result":"ERROR_NO_FROM"}`.
3. **Нэг алдаатай кейс — `transfer-neg`, мөр 12 (`~Frozen, Valid, Normal, Same`).**
   - **Setup:** `F1` данс Frozen; `B1` идэвхтэй; дүн 50,000₮-ийг шилжүүлэхэд хангалттай үлдэгдэл, лимиттэй.
   - **Оролт:** `POST /transfer {"from":"F1","to":"B1","amount":50000,"currency":"MNT"}`
   - **Oracle:** `200 OK`, `{"result":"ERROR_FROZEN"}`.

### B3. Greedy алгоритм ба дүгнэлт

Greedy аргаар дараагийн тестийн мөрийг сонгохдоо одоогоор хамрагдаагүй параметрийн хос утгуудыг аль болох олныг хамруулахыг зорьдог. Шинэ мөр нэмэх бүрд хамрагдсан хосуудыг шинэчилж, шаардлагатай хосын хамралт бүрдэх хүртэл сонголтоо үргэлжлүүлнэ. Ийм аргаар бүх боломжит Cartesian комбинацийг туршихаас цөөн мөрөөр pairwise хамралт гаргаж болно. Энэ нь эвристик арга тул бүх загварт математик абсолют хамгийн цөөн мөрийг олно гэж баталдаггүй.

Category-Partition нь параметрийн ангилал, бизнесийн утга, алдааны үр дүнг тодорхой болгож өгдөг. Энэ аргаар transfer функцийн үндсэн төлөөлөх утгууд болон давхар алдааны шалгах эрэмбийг тодорхойллоо. PICT нь browser болон transfer загваруудын хосын хамралтыг автоматжуулж, бүх комбинацийг жагсаахгүйгээр тестийн багц үүсгэлээ. Хязгаарлалтгүй загвар 13 мөр (happy path 1, нэг алдаатай 1), `IF` хязгаарлалттай загвар 12 мөр (happy path 0, нэг алдаатай 3), `~` загвар 16 мөр (happy path 2, нэг алдаатай 14, олон алдаатай 0) үүсгэсэн. Мөрийн тоо цөөн хувилбарууд олон алдааг нэг мөрөнд хольж, error masking-ийн улмаас дүнгийн алдаануудыг огт шалгахгүй байсан бол `~` загвар цөөн мөр нэмж бүх алдааны ангийг тусад нь, happy path-ийг хоёр валюттай шалгав. `~` ашиглах үед Missing-д зориулсан `IF` хязгаарлалт илүүц болохыг PICT-ийн гаралт ижил гарснаар баталгаажуулав.
