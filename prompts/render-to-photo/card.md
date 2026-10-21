# render-to-photo: фото комнаты из 3D-рендера

| Поле | Значение |
|---|---|
| Название | Фото комнаты из 3D-рендера без сдвига мебели |
| ID | `render-to-photo` |
| Версия | v1.0 |
| Статус | active |
| Владелец | Дмитрий Гайсин ([@dgmagic82](https://github.com/dgmagic82)) |
| Модель | GPT Image (ChatGPT, режим создания изображения) |
| Теги | `image`, `interior`, `3d-render`, `photoreal`, `furniture`, `geometry-lock` |
| Шаблон | [template.md](template.md) |
| Разбор v0 → v1.0 | [CLEAR.md](CLEAR.md) |
| Обновлено | 22.10.2026 |

## Цель

Получить фотореалистичное фото комнаты по кадру из Blender, где мебель стоит в реальных размерах, и не потерять эту точность. Модель меняет только материалы и свет. Сдвиг любого предмета больше 3 px считается браком.

Применять: визуализация расстановки мебели по реальной планировке, карточки «как это будет выглядеть у вас».
Не применять: когда нужно придумать интерьер с нуля или добавить декор. Шаблон прямо запрещает новые предметы.

## Как запускать

1. Новый чат в ChatGPT, режим создания изображения.
2. Прикрепить ровно один кадр. Фото товаров не прикладывать: по ним модель перерисовывает форму.
3. Подставить значения переменных в [шаблон](template.md) и отправить одним сообщением.
4. Сохранить результат под именем кадра и прогнать проверку (раздел «Критерии приёмки»).
5. Не прошло: заполнить `strict_note` значением из таблицы ниже и повторить. Не больше двух повторов, потом разбирать кадр.

## Переменные

| Переменная | Что подставлять | Пример |
|---|---|---|
| `room_type` | Тип комнаты, одно-два слова | `bedroom` |
| `room_finishes` | Стены, потолок, пол, плинтус, рама окна | `warm white matte painted walls and ceiling, light natural oak laminate floor with long planks and a subtle grain, white skirting boards, white PVC window frame` |
| `window_view` | Что за окном | `a soft-focus green courtyard with trees on a bright overcast day` |
| `lighting` | Источник и характер света | `soft natural daylight from the window` |
| `furniture_list` | По строке на предмет: что это и из чего. Только материалы, форму не описывать | см. пример ниже |
| `aspect_ratio` | Пропорции кадра, как у рендера | `3:2 landscape` |
| `strict_note` | Пусто при первом прогоне. При повторе: `The previous attempt changed the geometry. Treat the render as a strict tracing template: every edge of the furniture must stay on the same pixels. Only retexture and relight.` | пусто |

## Пример входа и выхода

**Вход.** Кадр [examples/bedroom_1.jpg](examples/bedroom_1.jpg) (1536 × 1024, рендер Blender) и значения из таблицы выше, плюс:

```
furniture_list:
- double bed with a bookshelf headboard: light grey-beige Sonoma oak laminate; white mattress, light warm-grey duvet, white pillows, smooth and slightly wrinkled cotton;
- two bedside tables: body in Sonoma oak, drawer fronts in whitewashed light grey wood with raised panels, antique silver metal handles;
- tall narrow wardrobe: Sonoma oak body, matte white doors with a horizontal oak band, satin steel handle.
```

**Выход шаблона** (что уходит в модель): [examples/bedroom_1_prompt.txt](examples/bedroom_1_prompt.txt).

**Ожидаемый выход модели:** фото той же спальни 3:2. Кровать, две тумбы и шкаф на своих местах, дуб Сонома и белые фасады, дневной свет из окна, за окном зелёный двор. Людей, надписей и новых предметов нет. Эталонное фото появится в `examples/` после первого принятого прогона.

## Критерии приёмки

- Для каждого предмета сдвиг не больше 3 px, и не меньше 70 % контуров рендера находят контур на фото в радиусе 3 px. Считает скрипт `photo_check.py` из проекта эксперимента (по ID-карте объектов). Без скрипта: наложить фото на рендер с прозрачностью 50 % и проверить, что края мебели совпадают.
- Пропорции 3:2, кадрирование не изменилось.
- Нет людей, текста, водяных знаков и предметов, которых нет в рендере.

## Тест-кейсы

| # | Что проверяем | Вход | Ожидаемый результат |
|---|---|---|---|
| 1 | Базовый сценарий | `bedroom_1.jpg`, значения из примера выше | Все 4 предмета (кровать, 2 тумбы, шкаф) проходят проверку сдвига и контуров. Отделка соответствует переменным. |
| 2 | Другая комната без правки текста шаблона | [examples/living_1.jpg](examples/living_1.jpg); `room_type` = `living room`; `furniture_list` = диван из серого велюра со строчкой, журнальный стол и ТВ-тумба из дуба Сонома, тонкий чёрный телевизор, кресло из серо-коричневой рогожки (полный текст в [examples/living_1_prompt.txt](examples/living_1_prompt.txt)) | Все 5 предметов проходят проверку. Экран телевизора выключен, на нём нет картинки. |
| 3 | Смена отделки и света не трогает геометрию | `bedroom_1.jpg`; `room_finishes` = `light sage green matte walls, white ceiling, dark walnut herringbone parquet, white skirting boards, black aluminium window frame`; `window_view` = `a city street at dusk with warm blurred lights`; `lighting` = `warm evening light, dusk outside` | Стены зелёные, пол тёмный ёлочкой, вечерний свет. Мебель на тех же пикселях, что в кейсе 1. Модель не дорисовала люстру, бра или лампу ради вечернего света. |

## История версий

| Версия | Дата | Что изменилось |
|---|---|---|
| v0 | 24.09.2026 | Личный промпт под одну квартиру: общий блок, два блока мебели, блок «если уехало», инструкция в том же файле. |
| v1.0 | 22.10.2026 | Рефакторинг по CLEAR: разделы TASK/CONTEXT/INPUT/OUTPUT, 7 переменных, правило геометрии в одном месте, скрытый контекст про проверку поднят в промпт, инструкция и критерии вынесены в карточку. |
