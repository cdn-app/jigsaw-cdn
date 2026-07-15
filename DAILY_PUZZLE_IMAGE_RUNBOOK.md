# Daily Puzzle AI Image Runbook

Tai lieu nay la runbook thuc thi de tao anh Daily Puzzle bang AI va import vao CDN. Doc cung voi `DAILY_PUZZLE_AI_PLAN.md`:

- `DAILY_PUZZLE_AI_PLAN.md`: nguon su that cho chu de theo ngay trong thang, rule sang tac va QA chi tiet.
- File nay: quy trinh lam viec tu luc chon ngay den luc anh live tren CDN.

Muc tieu: moi ngay co **1 anh AI** dep, dung chu de, dung 4:3, lam puzzle tot, va co duy nhat mot `PlayJigsaw.net` nam tu nhien trong canh. Khong dung Pixabay/Unsplash hay anh stock.

## Nguyen tac bat buoc

1. Truoc khi tao bat ky anh nao, tra bang `Day` trong `DAILY_PUZZLE_AI_PLAN.md`. `Day pool` cua day-of-month la bat buoc, **tru khi ngay do nam trong special override da ghi ro trong Plan**; khong tu freestyle sang day pool khac.
2. Trong day pool, phai chon `Theme lane` va subtheme theo seed thang/nam de tranh lap. Khong coi moi ngay la 1 chu de hep duy nhat.
3. Tao 1 anh/1 ngay. Khong tao nhieu anh neu anh dau da dat QA; chi reroll khi co loi ro rang.
4. Moi anh phai doi palette, anh sang va style mot cach hop ly so voi cac ngay gan nhat. Khong de nau/go chiem anh tru khi chu de bat buoc can go.
5. Uu tien do vat quen thuoc trong nha, bep, ban lam viec, do choi, phuong tien...; khong hoa my hoa canh cay/rung khi chu de khong yeu cau.
6. Khong co chu, logo, watermark, brand, poster, UI, caption hoac nhan noi bat. Ngoai le duy nhat la `PlayJigsaw.net`.
7. Khong bao gio ghi PAT/API key vao Markdown, prompt, log hay commit. Lay credentials tu cau hinh admin/local environment.

## Workflow chuan

### 1. Chon ngay va doc plan

Xac dinh date key `YYMMDD`, vi du 20/10/2026 la `261020`.

Tra `Day` trong bang chu de cua Plan, lay dung `Day pool`, roi chon 1 `Theme lane` + subtheme trong dung hang do. Vi du:

| Day | Day pool | Vi du lane/subtheme |
| --- | --- | --- |
| 17 | Books & Reading | reading nook / armchair, blank books, glasses, bookmark |
| 18 | Baking & Dessert Prep | cupcake tray / oven mitt, berries, batter |
| 19 | Kids Room & Toys | toy cars / plain blocks, cubby shelf, play mat |

Qua thang moi, uu tien doi lane truoc khi doi rieng style. Vi du Day 01 co the xoay giua kitchen prep, cookware corner, small appliances, sink and clean counter, colorful utensils.

Sau khi lay dung `Day pool`, kiem tra them section **Seasonal / holiday overlay va special override** trong Plan:

- Neu ngay nam trong special override, dung chu de dac biet cua dip do thay cho day pool thuong, nhung van giu household/puzzle-friendly va QA text/logo/delegate.
- Neu ngay nam trong dip le/mua vu lon, tron overlay vao day pool cua ngay do.
- Day pool van la xuong song; overlay chi them mau sac, props va atmosphere.
- Khong them chu le hoi vao anh, vi du khong viet "Halloween", "Christmas", "Happy New Year".
- Neu overlay lam anh qua poster/fantasy/hoa my, giam overlay va quay ve household theme.

### 2. Chon huong my thuat truoc khi viet prompt

Chon mot huong phu hop voi chu de va khac voi cac anh gan nhat:

- Style: 3D claymation, isometric 3D cartoon, whimsical watercolor, papercraft diorama, vector storybook, soft toy-like 3D, cozy digital painting.
- Palette: cool clean, bright kitchen, pastel textile, playful primary, fresh green, neutral pop, warm home co accent mau lanh.
- Anh sang: uu tien mid-tone, soft shadow, khong qua sang/catalog-like, khong qua toi.
- Bo cuc: landscape 4:3, chu the ro rang, nhieu manh chi tiet vua du cho puzzle, khong roi va khong co mot vung trong lon.

### 3. Viet prompt va tao anh

Dung built-in image generation. Prompt nen ngan gon nhung co du cac phan sau:

```text
Use case: stylized-concept.
Asset type: daily online jigsaw puzzle for DD Month YYYY.
Theme: <day pool tu Plan>; lane/scene: <theme lane + subtheme tu Plan>.
Style: <style>.
Palette/lighting: <palette va anh sang>.
Composition: polished landscape 4:3, puzzle-friendly, clear medium-sized objects.
Text: exactly one readable instance of "PlayJigsaw.net" as <cach the hien tu nhien>.
Constraints: no people neu khong can; no other readable text, logo, brand, watermark, UI, border, frame.
Avoid: misspelled PlayJigsaw.net, duplicate PlayJigsaw.net, overlay watermark, promotional text,
dominant brown, blown highlights, excessive empty space.
```

Khong can ep 1 batch phai co quota ky thuat delegate text. Moi anh chi can chon cach the hien phu hop nhat voi chinh canh do.

### 4. Rule `PlayJigsaw.net`

Phai co dung **mot** instance, dung chinh ta va dung case: `PlayJigsaw.net`.

No la chi tiet trong the gioi cua anh, khong phai watermark. Chon cach dat theo vat lieu, goc nhin va bo cuc, vi du:

- khac/dap chim tren gom, kim loai, go, da;
- theu, in muc mo hoac det vao vai khi be mat phu hop;
- viet phan/dong dau/son mo tren vat phu;
- nhan/tag chi khi no la lua chon tu nhien nhat, khong phai fallback mac dinh.

Bat buoc:

- Chu phai theo perspective va bieu dang cua be mat. Vat nghiêng thi chu nghiêng; be mat cong/gap/vai mem thi chu cong/meo tu nhien theo no.
- Kich thuoc **discoverable micro-detail** (muc tieu khoang 0.55%-0.9% chieu rong), low contrast, o vung phu/ria/canh, khong dat tren vat the chinh neu lam no thanh focal point.
- Full image khong duoc bi hut vao chu, nhung khi soi/zoom phai tim thay va doc duoc `PlayJigsaw.net`. Neu full view doc ro nhu label/watermark thi reject; neu zoom van khong thay hoac khong doc duoc thi cung reject.
- Tranh bien `PlayJigsaw.net` thanh label/sticker/tag/plaque rieng. Uu tien no nam tren canh/rim/lip/canh ben/can do phu, bam vat lieu va perspective; neu la tag/nhan thi phai rat nho, nghieng, bi hoa vao vat the va khong nam tren mat phang sang sach.
- Khong dung tag/nhan/plaque sang mau lam fallback mac dinh. Chi dung tag/nhan khi no rat nho, mau tiep, nghieng dung perspective va that su hop voi canh.
- Neu AI kho lam dung tren vai/giay gap, chon be mat on dinh hon nhu canh hop, canh sach, coaster, vat gom/kim loai phang.

Reject ngay neu chu dung ngang nhu overlay tren vat nghieng, thanh watermark/label/sticker/tag noi, qua sang, qua dam, qua lon, full image doc ro ngay, la diem nhin dau tien, bi sai chinh ta, bi trung hoac bi mat.

### 5. QA truoc khi import

Xem anh o hai muc:

1. Full image truoc: theme dung chua, bo cuc co dep va puzzle-friendly khong, palette/style co lap lai gan day khong, text co hut mat khong. Neu mat bi keo vao `PlayJigsaw.net` truoc chu the chinh thi reject.
2. Zoom/crop sau: `PlayJigsaw.net` co dung chinh ta, dung 1 lan, bam dung material/perspective khong; co chu/logo nao khac khong.

Checklist pass:

- Dung theme/subtheme cua ngay.
- 4:3 landscape; chu the khong bi crop xau.
- Co do vat va texture du cho puzzle, nhung khong qua roi.
- Khong text/logo/watermark khac.
- Delegate text dung rule, khong noi bat.
- Palette va style khac hop ly so voi anh gan nhat.
- Khong nguoi that/mat can canh neu chu de khong can.

Neu fail mot loi quan trong (text sai, text overlay, sai chu de, qua nhat mau, anh kem dep), reroll voi mot thay doi muc tieu, khong can tao hang loat variant.

## Chuan hoa file

Sau khi chon anh:

1. Crop/fit ve **1280 x 960** (4:3), dung Lanczos.
2. Convert sang **WebP quality 85**.
3. Kiem tra lai sau crop/nen vi chu the hoac delegate text co the bi cat/mat.
4. Dat ten:

```text
daily_puzzles/YYYY/MM/daily_YYMMDD_ai-<theme-slug>-<hash>.webp
```

Vi du:

```text
daily_puzzles/2026/10/daily_261019_ai-toy-storage-d5146d1662.webp
```

Muc tieu dung luong WebP: khoang 100KB-900KB. Khong chon anh qua mo hoac co artifact nen.

## Import CDN va manifest

Repo CDN hien tai: `cdn-app/jigsaw-cdn`.

Thu tu bat buoc:

1. Upload WebP moi vao `daily_puzzles/YYYY/MM/...`.
2. Cap nhat entry `YYMMDD` trong `daily_puzzles/manifest.json`.
3. Verify raw URL tra `HTTP 200`:

```text
https://raw.githubusercontent.com/cdn-app/jigsaw-cdn/main/daily_puzzles/<fileName>
```

4. Neu thay the ngay cu: chi xoa file cu **sau** khi file moi upload thanh cong va manifest moi da update.

`fileName` trong manifest phai bao gom ca `YYYY/MM/`, vi admin ghep URL theo field nay.

Metadata toi thieu nen co:

```json
{
  "fileName": "2026/10/daily_261019_ai-toy-storage-d5146d1662.webp",
  "source": "ai",
  "imageId": "ai-261019",
  "theme": "Toy Storage",
  "subtheme": "cubby shelf, toy cars, blocks without letters, plush basket, play mat",
  "style": "clean isometric 3D cartoon",
  "prompt": "final compact prompt summary",
  "negativePrompt": "other readable text, misspelled PlayJigsaw.net, duplicate PlayJigsaw.net, overlay watermark...",
  "model": "gpt-image-2 built-in imagegen",
  "qaStatus": "approved",
  "qaScore": 90,
  "qaChecks": {
    "theme": true,
    "composition": true,
    "delegateText": true,
    "perspective": true,
    "otherText": true,
    "palette": true
  },
  "width": 1280,
  "height": 960,
  "authorName": "AI Generated",
  "authorProfile": "https://playjigsaw.net"
}
```

## Cach dung admin

- Khi F5 `admin.html`, no phai mo o ngay/thang/nam hien tai.
- Dung nut previous/next month o header thay vi dropdown.
- Chon ngay de xem preview va metadata; Auto CDN de upload image da chon.
- Admin hien chi dung anh `source: "ai"`; khong dua Pixabay/Unsplash tro lai manifest.

## Hand-off checklist sau moi batch

- [ ] Tat ca ngay trong batch dung chu de tu Plan.
- [ ] Anh da QA full + zoom.
- [ ] Da chuyen WebP 1280x960 quality 85.
- [ ] Da upload image va update manifest.
- [ ] Raw URL cua tung anh HTTP 200.
- [ ] Da xoa file tam/QA sheet/script tam; giu lai final WebP trong repo.
- [ ] Khong co token, secret, duong dan credentials trong output/commit.

## Khi cai lai Codex

1. Mo repo `jigsaw-cdn`.
2. Doc file nay truoc, sau do doc `DAILY_PUZZLE_AI_PLAN.md` va tra day theme.
3. Dung skill `imagegen` cua Codex de tao tung anh bang built-in tool.
4. Lam dung workflow Generate -> QA -> WebP -> Upload -> Manifest -> HTTP 200.
5. Neu can credentials, nhap/cau hinh lai trong admin local; khong tim hay copy token tu tai lieu nay.
