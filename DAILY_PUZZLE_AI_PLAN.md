# Daily Puzzle AI Image Plan

> Quy trinh thuc thi, import CDN va hand-off: xem `DAILY_PUZZLE_IMAGE_RUNBOOK.md`.

## Muc tieu

Chuyen Daily Puzzle tu nguon anh stock Pixabay/Unsplash sang anh tao bang AI. `admin.html` se van giu cac phan dang lam tot hien tai:

- Chon ngay tren lich.
- Bulk Auto-Fill theo thang.
- Preview anh.
- Crop/resize/nen WebP bang canvas.
- Upload vao GitHub repo/CDN.
- Cap nhat `daily_puzzles/manifest.json`.
- Re-roll ngay da chon va xoa anh cu.

Phan can thay la nguon tao anh: thay `fetch Unsplash/Pixabay` bang `generate AI image`.

## Pipeline de tao va import anh

1. Admin chon thang/nam hoac chon mot ngay.
2. He thong tao prompt tu `dateKey`, `day`, `month`, `year`, theme cua ngay, subtheme, style preset, va seed.
3. Goi AI image provider de tao 1 hoac nhieu anh ung vien.
4. Chay QA gate cho tung anh ung vien.
5. Hien preview/picker neu tao thu cong, hoac auto chon anh dat diem QA cao nhat neu Bulk.
6. Tai blob anh ve browser.
7. Chay `convertToWebP(blob, maxWidth, quality, cropRatio)`.
8. Chay QA gate lan 2 tren anh WebP da crop/nen.
9. Dat ten file theo chuan CDN.
10. Upload WebP vao GitHub:
   - `daily_puzzles/YYYY/MM/daily_YYMMDD_ai-<themeSlug>-<hash>.webp`
11. Ghi metadata vao `manifest.json`.
12. Neu ngay do da co anh cu, xoa file cu sau khi upload thanh cong.

## Schema manifest de xuat

```json
{
  "260701": {
    "fileName": "2026/07/daily_260701_ai-forest-glade-a1b2c3.webp",
    "source": "ai",
    "imageId": "ai-a1b2c3",
    "theme": "Forest Worlds",
    "subtheme": "Mushroom village",
    "style": "3d claymation",
    "prompt": "final prompt used for generation",
    "negativePrompt": "text, watermark, logo, signature, people, gore, horror",
    "model": "configured-ai-provider-model",
    "seed": "2026-07-01-forest-glade",
    "originalUrl": "",
    "authorName": "AI Generated",
    "authorProfile": "",
    "qaStatus": "passed",
    "qaScore": 92,
    "qaChecks": {
      "aspectRatio": "passed",
      "minResolution": "passed",
      "fileSize": "passed",
      "textLogoWatermark": "passed",
      "composition": "passed",
      "duplicate": "passed"
    },
    "qaNotes": "",
    "downloadedAt": "2026-07-01T00:00:00.000Z"
  }
}
```

Ghi chu: co the giu cac field cu (`fileName`, `source`, `imageId`, `originalUrl`, `authorName`, `authorProfile`, `downloadedAt`) de website hien tai khong bi vo. Cac field moi chi la bo sung.

## Rule prompt chung

Moi prompt nen co 6 lop:

1. Subject: chu the chinh cua buc anh.
2. Environment: khong gian/boi canh.
3. Puzzle detail: nhieu vat the nho, mau sac ro, texture ro de ghep puzzle vui.
4. Composition: landscape 4:3, clear focal point, no cropped subject.
5. Style: consistent cute premium illustration/clay/3D/papercraft.
6. Safety: no logo, no watermark, no real brand, no readable UI overlay.

Template:

```text
Create a premium family-friendly jigsaw puzzle image: <subject> in <environment>,
with many delightful small details, clear shapes, rich color contrast, cozy lighting,
landscape composition, 4:3 aspect ratio, centered main subject, no cropped edges.
Style: <style>.
Avoid: photorealistic humans, real brands, logos, watermark, signature, gore, horror,
politics, religious symbols, copyrighted characters, readable overlay text.
```

## Diegetic text rule

Diegetic text la chu nam trong the gioi cua anh, vi du bang go, nhan chai, bien cua hang, sach mo tren ban. Với AI image, text de bi sai chinh ta, nen rule nen chat:

- Default: khong tao readable text.
- Neu can co bang hieu/sach/nhan do vat, dung "blank sign", "decorative symbols", "tiny unreadable glyphs".
- Khong dung text de truyen thong tin quan trong cua puzzle.
- Khong de chu, logo, watermark, UI text, poster, label lon nam ngoai canh.
- Neu muon seasonal vibe, dung hinh anh thay chu: la do cho mua thu, long den cho le hoi, tuyet cho mua dong.
- Neu bat buoc co text ngoai `PlayJigsaw.net`, chi cho 1-2 tu ngan, tieng Anh don gian, va phai review preview truoc khi publish. Khong dung cho bulk auto.

### PlayJigsaw.net delegate text

`PlayJigsaw.net` la ngoai le duy nhat duoc phep co readable text. Muc tieu khong phai lam watermark/quang cao, ma la mot chi tiet nho nam tu nhien trong canh.

- Bat buoc co dung 1 instance: `PlayJigsaw.net`.
- Chu phai dung chinh ta khi crop/zoom gan, khong thieu dau cham, khong them ky tu. O full image no chi nen la chi tiet rat nho, khong can doc ro ngay.
- Cach dat phai linh hoat theo canh va vat lieu, khong co kieu mac dinh.
- Co the la tag, nhan vai, nhan giay, bien dong nho, khac go, dap da/da thuoc, theu, viet phan, dong dau mo, in tren hop, hoac cach khac neu hop logic canh.
- Khong uu tien rieng "in chim/khac/dap"; chi chon cach nao tu nhien nhat voi vat the trong anh.
- Moi anh chi chon 1 cach the hien `PlayJigsaw.net` phu hop nhat voi vat lieu, goc nhin va bo cuc cua chinh canh do. Co the khac chim, dap noi, theu, in muc mo, viet phan, dong dau, son nho, in tren vat the hoac dung tag/nhan; khong mac dinh bam vao bat ky mot loai nao.
- Tag/nhan chi dung khi no la lua chon tu nhien nhat trong canh, khong phai fallback mac dinh. Khong ep quota hay co gang nhieu ky thuat trong cung mot anh/batch.
- Khi chon cach the hien, uu tien logic vat the va tham my cua anh; su da dang la ket qua cua viec chon dung theo tung canh, khong phai muc tieu may moc can dat bang moi gia.
- Chu phai bam theo phoi canh va be mat vat the: neu vat the nghieng thi chu/ngan tag phai nghieng theo; neu be mat cong/gap/vai mem thi chu phai hoi meo/chim theo vat lieu. Khong duoc nam ngang tuyet doi nhu overlay neu mat phang ben duoi dang nghieng.
- Neu khong chac AI render dung phoi canh tren vai/giay bi gap, uu tien dat `PlayJigsaw.net` tren vat phang ro mat phang hon: tag treo rieng, nhan may vuong, canh hop, coaster, canh sach, hoac nhan tren be mat it bien dang.
- Dat tren vat phu hoac chi tiet phu, uu tien vung ria/canh/nen phu; tranh vung trung tam, tranh vat the chinh, tranh be mat lon doi dien camera.
- Kich thuoc muc tieu: **discoverable micro-detail**, khoang 0.55%-0.9% chieu rong anh. Nho hon muc nay de mat hut thi fail; lon/ro nhu label hoac watermark thi cung fail.
- Do noi/contrast: **thap**, cung chat lieu/tong mau voi vat the; nen giong khac/dap/theu/in mo hon la chu in ro. Khi full view khong duoc hut mat; khi soi/zoom phai tim thay va doc duoc.
- Tranh dat `PlayJigsaw.net` tren tag/nhan trang sang, plaque noi, sticker, bien nhan rieng, mat giay/trang/notebook, cup wrap, hoac mat phang lon sach. Uu tien canh/rim/lip/canh ben/can do phu o ria anh, bam dung perspective/curvature/vat lieu. Tag/nhan chi chap nhan khi cuc nho, nghieng, low contrast, bi hoa vao vat the va khong phai diem nhin.
- Reject neu giong overlay, watermark, sticker quang cao, tag/label sang qua noi, chu qua sach/qua dam, nam dung diem nhin dau tien, la text noi bat nhat trong anh, full image doc ro ngay, hoac chu khong xoay/nghieng/meo theo phoi canh va chat lieu cua vat the ben duoi.
- QA 2 buoc: xem full image truoc; neu mat bi keo vao chu trong 1 giay dau thi reject/regenerate. Sau do moi crop/zoom de check dung chu va dung 1 instance.

Negative prompt nen luon co:

```text
other readable text, misspelled PlayJigsaw.net, duplicate PlayJigsaw.net, overlay watermark,
signature, logo, brand name, poster text, caption, UI, border, frame, human face close-up,
scary, violent, gore
```

## Rule anh cho jigsaw

- Ty le mac dinh: 4:3 landscape.
- Output cuoi: WebP, max width 1280, quality 85%.
- Nen co foreground/midground/background ro.
- Nen co 1 focal point chinh va nhieu chi tiet phu.
- Tranh anh qua toi, qua don sac, qua nhieu vung blur.
- Tranh bo cuc chi co mot vat the lon tren nen tron.
- Tranh anh co qua nhieu chi tiet sieu nho kho nhin tren mobile.
- Tranh mat nguoi that/nhan vat noi tieng/copyrighted characters.
- Palette nen thay doi theo ngay, khong de ca thang cung mot mau.

## QA gate sau khi tao anh

Anh AI khong duoc upload thang len CDN. Moi anh phai qua 2 vong check:

1. Check tren anh goc provider tra ve.
2. Check lai sau khi crop/resize/nen WebP, vi crop co the cat mat chu the hoac lam anh qua mo.

### Automated checks

- File hop le:
  - Anh load duoc trong browser.
  - Width/height doc duoc.
  - Khong bi zero-byte, broken image, transparent/blank.
- Resolution:
  - Anh goc nen toi thieu 1024px o canh ngang.
  - Anh cuoi nen la 4:3 hoac dung cropRatio da chon.
- File size:
  - WebP nen trong khoang 100KB-900KB.
  - Qua nho co the bi mo/it chi tiet; qua lon can nen lai.
- Aspect/crop:
  - Chu the chinh khong bi cat dau/cat vien.
  - Focal point nam gan trung tam, khong sat mep.
- Visual density:
  - Co du chi tiet cho puzzle, khong phai nen tron voi 1 vat the duy nhat.
  - Khong qua roi, khong qua nhieu chi tiet li ti khong doc duoc tren mobile.
- Safety text/logo:
  - Reject neu thay watermark, signature, logo, brand, UI, caption, poster text.
  - Reject neu co text lon, text sai chinh ta, hoac text la noi dung quan trong.
  - Rieng `PlayJigsaw.net`: phai co dung 1 instance, dat tu nhien trong canh, doc duoc khi zoom, nhung khong duoc noi bat o full image; phai bam dung phoi canh/be mat vat the, khong duoc trong nhu chu overlay dan ngang len anh.
- People/identity:
  - Reject nguoi that, mat nguoi can canh, nguoi noi tieng, nhan vat co ban quyen.
- Duplicate:
  - Khong dung lai cung `imageId`.
  - Nen tinh `promptHash` va `imageHash`/`perceptualHash` de tranh anh qua giong ngay/thang truoc.
- Theme match:
  - Anh phai khop theme/subtheme cua ngay.
  - Neu prompt la forest ma anh ra city/space thi reject.

### Manual review checklist

Truoc khi publish full thang, admin nen co che do `Review before upload`:

- Anh co dep va hop lam puzzle khong?
- Co bi chu/logos/watermark khong, ngoai 1 `PlayJigsaw.net` dat dung rule?
- `PlayJigsaw.net` co bi noi bat qua, giong overlay/tag quang cao, hoac hut mat hon chu the chinh khong?
- Chu the co bi crop sai khong?
- Anh co qua toi, qua mo, qua nhat mau khong?
- Co giong anh da dung trong thang nay khong?
- Co dung theme/subtheme khong?
- Co noi dung nhay cam, dang so, chinh tri, ton giao, thuong hieu that khong?

### QA scoring

Moi anh ung vien co diem 0-100:

- 20 diem: dung theme/subtheme.
- 20 diem: bo cuc/focal point tot.
- 20 diem: chi tiet puzzle tot.
- 15 diem: mau sac/anh sang tot.
- 15 diem: khong text/logo/watermark ngoai 1 `PlayJigsaw.net` dat dung rule.
- 10 diem: khong trung lap voi anh cu.

Publish gate:

- `qaScore >= 85`: auto pass neu Bulk khong bat review.
- `70 <= qaScore < 85`: can review thu cong.
- `< 70`: reject va regenerate.

### Regenerate / fallback rule

Neu anh khong dat:

1. Thu lai cung subtheme voi style khac.
2. Thu lai cung day pool nhung doi theme lane/subtheme khac.
3. Giam do phuc tap prompt neu anh bi loi hoac qua roi.
4. Neu 3 lan fail, danh dau ngay do `needsReview` va khong overwrite anh cu.

Quan trong: neu ngay da co anh cu dang live, chi xoa anh cu sau khi anh moi upload va manifest update thanh cong.

## Lich 31 ngay lap lai hang thang

Moi ngay la 1 **day pool**, khong phai 1 chu de hep hay 1 buc/canh co dinh. Moi day pool co 4-5 **theme lanes** than thuoc, va moi lane lai co nhieu subtheme/object detail. Khi qua thang moi, van dung ngay do nhung seed theo nam/thang se chon lane + subtheme + style + palette khac de anh khong trung noi dung.

Vi du Day 01:

- 2026-07-01 co the la `Kitchen Prep / cutting board + vegetables / watercolor`.
- 2026-08-01 co the la `Cookware Corner / pots + utensils / storybook illustration`.
- 2026-09-01 co the la `Small Appliances / toaster + kettle / soft 3D`.
- 2027-07-01 co the la `Sink & Clean Counter / dish rack + sponge / papercraft`.

Nhu vay mùng 1 hang thang van co "tinh cach" rieng de player nhan ra lich xoay vong, nhung anh va noi dung khong lap y chang.

Moi lan tao anh, chon theo thu tu: `Day -> Day pool -> Theme lane -> Subtheme -> Style/Palette`. Neu thang truoc da dung mot lane gan giong, uu tien doi lane truoc.

| Day | Day pool | Theme lanes / subthemes |
| --- | --- | --- |
| 01 | Kitchen & Cooking | kitchen prep; cookware corner; small appliances; sink and clean counter; colorful utensils |
| 02 | Fruits & Snacks | fruit bowl; snack tray; smoothie counter; lunch fruit prep; picnic fruit box |
| 03 | Morning Food | breakfast table; cereal and milk; toast and jam; pancake plate; juice and fruit |
| 04 | Living Room | sofa cushions; coffee table; rug and lamp; blanket basket; TV-free cozy corner |
| 05 | Bedroom & Sleep | fresh bedding; bedside table; quilt stack; slippers and rug; morning window |
| 06 | Laundry & Clothes Care | laundry basket; folded towels; clothesline; ironing corner; drying rack |
| 07 | Desk & Paper Goods | notebooks; pencil cup; envelopes; planner desk; craft desk supplies |
| 08 | Sewing & Textiles | sewing basket; fabric swatches; buttons; embroidery hoop; mending kit |
| 09 | Handmade Craft | pottery mugs; clay tools; ceramic tray; brush roll; small craft shelf |
| 10 | Home Tools & Repair | toolbox; pegboard tools; paint roller; screws and tape; small repair nook |
| 11 | Bathroom & Self Care | towel shelf; sink counter; soap dish; toothbrush cup; bath basket |
| 12 | Food Storage | open fridge; pantry containers; meal prep boxes; jars with blank labels; grocery sorting |
| 13 | Pet Corner | cat bed; dog basket; food bowls; pet toys; grooming corner |
| 14 | Balcony & Small Garden | watering can; herb pots; seedling tray; balcony mat; small plant shelf |
| 15 | Entryway & Going Out | shoe rack; umbrellas; tote bag; keys bowl; hats and scarves |
| 16 | Table Games & Family | board game; tokens and dice; puzzle pieces; card table without readable text; snack bowl |
| 17 | Books & Reading | reading nook; blank books; magazines without text; reading glasses; bookmark |
| 18 | Baking & Dessert Prep | mixing bowl; cupcake tray; oven mitt; rolling pin; berries and batter |
| 19 | Kids Room & Toys | cubby shelf; toy cars; plain blocks; plush basket; play mat |
| 20 | Drinks & Small Treats | tea set; cookies; fruit plate; folded napkin; cocoa or warm drink tray |
| 21 | Home Office & Tech | laptop with blank screen; mouse pad; headphones; blank sticky notes; cable organizer |
| 22 | Lunch & Picnic | sandwiches; lunch boxes; water bottle; carrot sticks; cloth napkin |
| 23 | Closet & Wardrobe | folded clothes; hangers; scarf; socks; storage baskets |
| 24 | Cleaning & Home Care | cleaning caddy; blank spray bottles; sponge; gloves; microfiber cloth |
| 25 | Coffee & Cafe Corner | coffee maker; mugs; coasters; beans; milk pitcher and spoon cup |
| 26 | Bath & Fresh Towels | hamper; fresh towels; bath mat; slippers; washcloth stack |
| 27 | Art Supplies | watercolor palette; brushes; blank sketchbook; water jar; paint cloth |
| 28 | Aquarium & Small Pets | fish tank; aquarium net; blank fish food jar; pebbles; water plants |
| 29 | Music & Listening | guitar or ukulele; headphones; blank music stand; record sleeves without text; small speaker |
| 30 | Pantry & Dry Goods | jars without labels; pasta and rice; bowls; measuring cups; folded cloth |
| 31 | Everyday Mixed Special | seasonal home table; best-of-household remix; celebration snacks; weekend hobby mix; mystery household corner |

Rule bat buoc khi tao anh:

- Truoc khi tao bat ky ngay nao, phai tra `Day` trong bang tren de lay dung `Day pool`.
- Khong duoc tu y doi sang day pool khac theo batch. Chi duoc chon `Theme lane` va `Subtheme` ben trong day pool cua ngay do.
- Moi thang phai uu tien doi lane/subtheme/object detail/palette/style/anh sang de tranh trung lap, nhung van giu dung tinh cach chung cua ngay.
- Neu day pool cua ngay do da bi dung qua gan day va tao cam giac hao hao, chon lane xa nhat trong pool truoc khi doi palette/style.
- Neu can doi theme cua mot ngay, phai sua bang Plan truoc roi moi tao anh.

## Seasonal / holiday overlay va special override

Binh thuong moi ngay tao anh theo bang **Lich 31 ngay lap lai hang thang**. Neu ngay do nam trong mot dip le/mua vu lon, them mot **seasonal overlay** vao chu de de anh hop thoi diem hon.

Nguyen tac:

- Overlay khong thay toan bo rule goc; no chi "nhuom" chu de ngay do bang vat dung, mau sac va khong khi cua dip le.
- Van giu do vat than thuoc trong nha, khong bien thanh poster/le hoi ngoai duong.
- Mot so dip ngan, co tinh bieu tuong manh, duoc phep dung **special override** rieng thay vi day pool thuong. Khi do anh van phai household/puzzle-friendly, nhung chu de chinh la dip do.
- Khong them chu le hoi trong anh. Vi du khong viet "Halloween", "Christmas", "Happy New Year".
- Van cam logo, brand, nhan vat ban quyen, gore/horror, religious symbols neu khong duoc yeu cau ro.
- `PlayJigsaw.net` van theo rule micro-detail, khong duoc thanh watermark hoac label noi.
- Neu seasonal overlay lam anh qua hoa my, qua nhieu cay coi, qua fantasy, thi giam overlay va quay ve household theme.

Special override hien tai:

| Time window | Special series | Direction |
| --- | --- | --- |
| Feb 10 - Feb 15 | Valentine's Day special | Duoc bo qua day pool 10-15 de tao mini-series Valentine rieng: gift wrapping table, chocolate/sweets prep, cozy tea/dessert for two without people, handmade blank card craft table, Valentine centerpiece/dessert table, after-Valentine cozy cleanup/gift basket. Khong co chu "Valentine", "Love", slogan, couple/people, brand/logo. |
| Oct 16 - Oct 31 | Halloween special | Duoc bo qua day pool 16-31 de tao mini-series Halloween household rieng: candy/game table, spooky reading nook, pumpkin baking, toy shelf, cocoa tray, home office, lunch boxes, costume closet, cleaning caddy, coffee corner, bath towels, art desk, aquarium, music corner, pantry, finale party table. Playful/cozy, khong gore/horror, khong chu Halloween/slogan. |

Seasonal overlay de xuat:

| Time window | Event / season | Overlay direction |
| --- | --- | --- |
| Jan 01 - Jan 03 | New Year | cozy reset, clean desk, fresh blank notebook/paper, snacks, warm drinks, confetti shapes without text |
| Jan 20 - Feb 20 | Lunar New Year / Tet-inspired optional | red/gold accents, fruit tray, tea set, envelopes without text, home cleaning/prep; avoid readable calligraphy |
| Feb 10 - Feb 15 | Valentine's Day | **special override**; see table above |
| Mar 10 - Mar 20 | Spring refresh | home cleaning, fresh laundry, light green/yellow palette, seedlings/herbs but still household |
| Mar 14 - Mar 17 | St. Patrick's Day optional | green accents, clover-like shapes, cozy kitchen/snack scene; no alcohol focus, no text |
| Late Mar - Apr | Easter / spring holiday optional | pastel eggs as decor/snacks, basket, breakfast table, craft desk; no religious symbols |
| May 01 - May 12 | Mother's Day season optional | breakfast tray, flowers as small home decor, tea set, handmade blank card, cozy kitchen/bedroom |
| Jun 10 - Jun 20 | Father's Day season optional | tools, hobby desk, coffee corner, small repair nook, blank card |
| Jul 01 - Jul 05 | Independence Day / summer picnic optional | red/white/blue palette, picnic, drinks, fruit, home table; no flags/text if not needed |
| Aug 01 - Aug 20 | Back-to-school / study season | notebooks, desk supplies, lunch boxes, backpacks without text, organized study corner |
| Sep 01 - Sep 10 | Labor Day / late summer optional | picnic, home projects, tool corner, laundry reset, simple summer colors |
| Oct 01 - Oct 15 | Cozy autumn | warm drinks, blankets, pantry, baking, orange/teal/purple accents, pumpkins as household decor |
| Oct 16 - Oct 31 | Halloween | **special override**; see table above |
| Nov 01 - Nov 10 | Post-Halloween autumn reset | pantry, cleaning, warm drinks, cozy living room, soft autumn palette |
| Nov 15 - Nov 30 | Thanksgiving / harvest season | kitchen prep, pie/baking, harvest vegetables, family table without people, warm home palette; no readable holiday text |
| Dec 01 - Dec 15 | Winter cozy | cocoa, blankets, baking, lights, home decor, cool blue/cream with warm accents |
| Dec 16 - Dec 26 | Christmas / winter holiday | tree/ornaments/lights/gift wrap without text, cookies, cocoa, cozy room; no religious symbols unless explicitly requested |
| Dec 27 - Dec 31 | Year-end / New Year's Eve | celebration snacks, hobby table, blank notebook, confetti shapes, cozy reset; no text/countdown numbers |

Khi tao prompt:

1. Lay `Day pool` theo ngay trong thang.
2. Kiem tra ngay/thang co seasonal overlay hoac special override khong.
3. Neu la special override, dung chu de dac biet cua dip do thay cho day pool thuong, nhung van giu style household/puzzle-friendly va QA text/logo/delegate.
4. Neu chi la overlay, tron overlay vao subtheme. Vi du:
   - Day 18 Baking vao Dec 20 => winter holiday cookies/cocoa baking counter.
   - Day 10 Home Tools vao Jun Father's season => small repair nook/gift-like blank card, khong text.
   - Day 27 Art Supplies vao Oct 25 => Halloween craft table voi pumpkins/candy decor, playful not horror.
5. Neu overlay va day pool xung dot, giu day pool la xuong song, overlay chi la mau/phu kien.
6. QA them: seasonal dung dip nhung khong lam mat tinh than household/puzzle-friendly.

## Palette va style rotation cho tung ngay

Khong dung 1 tong mau lap lai qua nhieu ngay. Palette/style la bien so rieng, khong duoc dong nhat ca thang.

Palette pool:

- Bright kitchen: mint, coral, lemon, white, sky blue.
- Cool clean: aqua, white, lavender, soft gray, pale yellow.
- Warm home: peach, cream, teal, soft orange, muted blue.
- Playful primary: cobalt, tomato red, sunshine yellow, mint, white.
- Pastel textile: lavender, teal, blush pink, butter yellow, cream.
- Fresh green accent: lime, aqua, white, fruit red, sky blue.
- Neutral pop: cream/white base with 2-3 strong accent colors.

Palette rules:

- Khong de mau nau/go chiem anh tru khi theme can vat lieu go; neu co go thi chi la accent.
- Khong lap cung palette lien tiep qua 2-3 ngay.
- Delegate text `PlayJigsaw.net` phai cung tone vat the, low-contrast, nho, nam vung phu; full image khong duoc bi hut mat vao text.
- Van phai doc duoc khi crop/zoom, dung chinh ta, dung 1 instance.
- QA delegate text theo tung anh: cach the hien co hop vat lieu, goc nhin va bo cuc khong; khong dung quota ky thuat theo batch.

## Style rotation

Dung style theo seed de khong bi lap:

- 3D claymation cute.
- Isometric 3D cartoon.
- Whimsical watercolor.
- Papercraft diorama.
- Vector storybook illustration.
- Soft toy-like 3D render.
- Cozy digital painting.

Cong thuc goi y:

```js
lane = DAY_POOLS[day].lanes[(month + year + day) % lanes.length]
subtheme = lane.subthemes[(month * 3 + year + day) % subthemes.length]
style = STYLES[(day + month + year) % STYLES.length]
variant = (year * 10000 + month * 100 + day)
```

## Ap dung vao admin.html

1. Sidebar credentials:
   - Bo `Unsplash Access Key`.
   - Bo `Pixabay API Key`.
   - Them `AI Image API Key`.
   - Them tuy chon provider/model neu can.

2. Auto tab:
   - Doi `Search Keywords` thanh `AI Prompt`.
   - Doi button `Unsplash/Pixabay` thanh:
     - `Generate AI`
     - `Generate Variants`
   - Giu `Upload File` cho truong hop manual override.

3. Preset:
   - Thay `PRESET_KEYWORDS` bang `DAILY_THEME_PLAN`.
   - Doi `generateCartoonPrompt(...)` thanh `generateDailyAiPrompt(day, month, year, index)`.

4. Bulk modal:
   - Bo checkbox `Unsplash API`, `Pixabay API`.
   - Them:
     - `Images per day` default 1.
     - `Style rotation` on/off.
     - `Review before upload` optional.
     - `Overwrite & Reset All Days`.

5. Bulk execution:
   - Trong `executeBulkFill(...)`, thay `fetchImageFromSource(...)` bang `generateAiImage(...)`.
   - Sau khi co image URL/base64/blob, tiep tuc dung lai:
     - `convertToWebP(...)`
     - GitHub upload
     - manifest update
     - cleanup old file

6. Metadata:
   - `source: "ai"`
   - `authorName: "AI Generated"`
   - `originalUrl: ""`
   - Luu prompt/style/theme/subtheme/seed/model de audit va re-roll.

7. UI preview:
   - Hien prompt da dung.
   - Hien theme/subtheme.
   - Hien nut `Copy Prompt`.
   - Hien nut `Re-generate Same Theme`.
   - Hien `QA status`, `QA score`, va danh sach check fail/pass.
   - Hien nut `Approve & Publish`, `Reject`, `Regenerate`.

8. QA implementation:
   - Them ham `validateGeneratedImage(blob, metadata)`.
   - Them ham `validateCompressedImage(webpBlob, metadata)`.
   - Them ham `scoreImageCandidate(candidate, qaChecks)`.
   - Them field `qaStatus`, `qaScore`, `qaChecks`, `qaNotes` vao manifest.
   - Bulk mode chi auto-publish anh `qaStatus === "passed"`.
   - Anh `needsReview` khong duoc overwrite anh live.

## Thu tu implement de an toan

1. Tao constants `DAILY_THEME_PLAN`, `STYLE_PRESETS`, `NEGATIVE_PROMPT`.
2. Tao `generateDailyAiPrompt(day, month, year, index)`.
3. Them field AI key vao credentials/localStorage.
4. Them UI Generate AI cho tung ngay.
5. Viet ham `generateAiImage(prompt, options)`.
6. Viet QA gate `validateGeneratedImage`.
7. Cho manual generate 1 anh hoat dong truoc.
8. Test QA tren anh pass/fail mau.
9. Sau do moi sua Bulk Auto-Fill Month.
10. Cap nhat manifest metadata.
11. Test voi 1 ngay rieng.
12. Test bulk 2-3 ngay.
13. Moi chay full 1 thang.

## Test cases can co

- Anh dep dung theme: pass.
- Anh co watermark/text/logo: reject.
- Anh bi crop dut chu the sau WebP: reject sau compression QA.
- Anh qua nho/vo/blank: reject.
- Anh dung theme nhung qua giong anh da co: reject duplicate.
- Bulk fail nua chung: cac ngay pass van giu draft, ngay fail vao `needsReview`, khong xoa anh cu.
- Manifest update fail: anh cu van duoc giu, log loi ro rang.

## Dieu can xac nhan truoc khi code

- AI image provider se dung la gi.
- Provider tra anh dang URL, base64, hay blob.
- Co can tao nhieu hon 1 anh/ngay mac dinh khong.
- Website frontend Daily Puzzle hien doc `manifest.json` can field nao bat buoc.
- Co can gan puzzle theo timezone nao cho user toan cau khong.
- Co muon bat buoc review thu cong cho 7 ngay dau tien truoc khi cho auto-publish khong.
