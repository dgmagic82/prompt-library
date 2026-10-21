# render-to-photo v1.0: шаблон

Тело промпта на английском: GPT Image точнее держит геометрию по английской инструкции.
Переменные в `{{двойных скобках}}`, описание каждой в [card.md](card.md#переменные).

```
TASK
Turn the attached 3D render of a {{room_type}} into a photorealistic photo of the same room. Change only surfaces and light.

CONTEXT
The render is an exact model of a real room with furniture at real sizes. The photo will be checked against it edge by edge.

INPUT
- Image: the attached render, the only visual reference.
- Room finishes: {{room_finishes}}
- Through the window: {{window_view}}
- Light: {{lighting}}
- Furniture finishes (materials only, shapes come from the render):
{{furniture_list}}

OUTPUT
One photorealistic image, {{aspect_ratio}}, like a professional photo of the real room: realistic materials, soft and contact shadows, ambient occlusion in corners, subtle reflections, slight grain, straight verticals. No people, text or watermark.
Keep exactly as in the render: camera, framing, perspective, horizon (no crop, zoom or rotation); walls, corners, doors, windows; every object's outline, position, size, proportions and details (drawers, shelves, handles, legs, cushions, seams). Add, remove, move or reshape nothing.
{{strict_note}}
```
