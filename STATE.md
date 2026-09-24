# STATE — boxing

- **Игра:** бокс 1×1, three.js r169 (вендорено в `lib/`), один `index.html`, GitHub Pages: https://zeghreit.github.io/boxing/
- **Стадия:** v0.2 (2026-09-24) — все ассеты AssetForge в игре. Если GLB не загрузился, игра падает обратно на примитивы.
- **Механика:** джеб/кросс/хук/аппер, блок, уклоны L/R, нырок, контры, выносливость, ИИ 3 уровня, нокдауны/счёт/ТКО, 3 раунда, тач под iPhone, `?mute`.

## Версии и обновление на телефоне (как в Kubik)
- Версия — текст в `<div class="brand">Бокс <span>…</span></div>` в меню. **При каждом пуше index.html менять её** (новое — число, фикс — буква: 0.2d → 0.2e → 0.3).
- `checkForUpdate()` при открытии и возврате в приложение читает index.html мимо кэша; другая версия → плашка сверху, тап = перезагрузка с `?v=`. Модели грузятся с `?v=<версия>`.
- После пуша дождаться сборки Pages (API `pages/builds/latest` = HEAD) и дать автору ссылку `https://zeghreit.github.io/boxing/?v=<версия>`. Сборка Pages может упасть без причины — перезапуск `POST /pages/builds`.

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
8. Клипы Mixamo бывают в разной стойке: «Dodging To The Right» — правша-вперёд, при кроссфейде из боксёрского idle руки менялись местами (жалоба автора). Проверка стойки клипа — `scratch/_stancecheck.py` (какая рука/нога ведущая в первом и последнем кадре). Уклоны L/R заменены процедурным наклоном позвоночника поверх idle. Mirror в Mixamo тоже переворачивает стойку.
9. Клипы шагов Mixamo («Boxing Step Forward/Backward», «Side Step Walk») разваливают стойку: перчатки 0.45–0.53 м от головы против 0.32 в idle (`scratch/_guardcheck.py`). Решено слоями в three.js: шаги только для таза и ног, корпус/руки/голова из idle; вес верхнего слоя = 1 − сумма весов полных клипов (иначе на переходах руки проваливаются в bind-позу).

10. (0.2e) Общий аудит клипов (`scratch/_clipaudit.py` — замеры + фронтальный контактный лист; `_scratch/_anim_audit.js` — замер в игре по всем состояниям): переходы в клипы из других наборов ломали стойку. Решение — маски по группам костей (legs/core/armL/armR): шаги=ноги; hit/hitBig/duck=ноги+корпус; jab/hook/upper=всё кроме правой руки; cross=всё кроме левой; остальное из idle-стойки. Вес стойки считать ПОСЛЕ `mixer.update(dt)` и затем `update(0)` — иначе в кадре смены клипа новый клип числится с весом 1 (fade-in ещё не начался) и руки на кадр падают в позу рига. Итог: нерабочая рука ≤0.29 м от головы во всех состояниях, всплесков 0.

## Дальше
- Фидбек с iPhone: читаемость, FPS с двумя скинированными моделями, баланс.
- Опционально: свои клипы нырка (Boxing Dodge Advance длинный, играет ×3.9), отдельный hit для хука/аппера.
