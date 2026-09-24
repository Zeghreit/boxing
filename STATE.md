# STATE — boxing

- **Игра:** бокс 1×1, three.js r169 (вендорено в `lib/`), один `index.html`, GitHub Pages: https://zeghreit.github.io/boxing/
- **Стадия:** v0.2 (2026-09-24) — все ассеты AssetForge в игре. Если GLB не загрузился, игра падает обратно на примитивы.
- **Механика:** джеб/кросс/хук/аппер, блок, уклоны L/R, нырок, контры, выносливость, ИИ 3 уровня, нокдауны/счёт/ТКО, 3 раунда, тач под iPhone, `?mute`.

## Ассеты (AF-прогон 2026-09-24)
| Ассет | Run / Tripo UUID | Итог в игре |
|---|---|---|
| boxer_player | `boxerp_r01` / 5a71aec2-6c15-4704-a732-8f141121db9c | `models/boxer_player.glb` 2.5 МБ, 9230 тр., 33 кости, 18 клипов |
| boxer_enemy | `boxere_r01` / 6617887f-e222-4129-ad05-67b90e70e1e9 | `models/boxer_enemy.glb` 2.5 МБ, 8994 тр., 33 кости, 18 клипов |
| corner_post | a6b0e456-ae02-4bf3-9768-372616cb1f6c | `models/corner_post.glb` 1990 тр.; синий/белый — перекраска текстуры в JS |
| stool | 3160c6db-810e-4b2c-be49-7dd8c2d7f65e | `models/stool.glb` 2152 тр. |
| bucket | 56692e2f-30e7-4f1e-abdc-869344afb123 | `models/bucket.glb` 2187 тр. |

Пайплайн бойца: Gemini-концепт → square_crop → Smart Mesh P2.0 Quad (polycount 4400) → Texture 8K → PBR → OBJ 1.8 м → Mixamo авториг → 18 клипов Mixamo → Blender-сборка GLB (base 2K jpg, normal + ORM 1K, кадр попадания каждого клипа в `models/*.json`).
Клипы: idle, jab, cross, hook, upper, block, slipL/R (Mirror), duck, hit, hitBig, ko, getup, win, stepF/B/L/R (In Place).
В игре: окно удара подгоняется под кадр попадания клипа (`impact`), клип ускоряется до окна логики.

Скрипты (assetforge/scratch): `boxing_to_obj.py`, `boxing_prop_glb.py`, `boxing_assemble.py`, `_asm_build.ps1`, `_mixamo_rig_now.py`, `_mixamo_clips.py` (выбор клипа по описанию), `_boxing_chain.ps1`.

## Находки для AF
1. Автоматный Chrome падал «Page crashed» ~8 раз за день, один раз умер целиком (порт 9333). Mixamo-поиск «head hit» роняет вкладку 3 из 3 раз.
2. `tripo_pbr` генерирует PBR, но не скачивает (Export) — забор отдельно: `tripo_open_asset` + `tripo_download`.
3. `tripo_image_to_3d` дважды не дождался нового ассета, хотя генерация прошла и кредиты списаны — ассет найден на /assets.
4. Mixamo поменял UI: новый шаг **Orient**, новый текст **Review** («Press Next to confirm»), новый модал **Change character** («Proceed with this new character?» — без ответа риг молча выбрасывается), шапка «Log in / Sign up» → `signed_in` врал. Всё поправлено в `assetforge/providers/mixamo.py` (не закоммичено; сервер подхватит после перезапуска Claude).
5. Mixamo переставляет выдачу поиска между вызовами — клипы выбирать по описанию, не по индексу.
6. Клипы без скина, скачанные на другом Mixamo-персонаже, нормально ложатся по именам костей.
7. Расширение Claude in Chrome «AssetForge» = тот же Chrome, что и очередь AF; параллельно не трогать.

## Дальше
- Фидбек с iPhone: читаемость, FPS с двумя скинированными моделями, баланс.
- Опционально: свои клипы нырка (Boxing Dodge Advance длинный, играет ×3.9), отдельный hit для хука/аппера.
