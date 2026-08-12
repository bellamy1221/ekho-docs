Да. Для Ekho тебе сейчас надо строить **насмотренность не только на визуал, а на 4 слоя: UX → UI → interaction → implementation**.
## Наш стандартный список ресурсов
### 1. Реальные продукты / UX — главное
1. **Mobbin** — №1. Реальные экраны и целые flows: onboarding, search, tables, settings, payments, empty states и т.д. ([mobbin.com][1])
2. **Page Flows** — смотри не отдельный экран, а **как пользователь проходит сценарий целиком**. ([pageflows.com][2])
3. **Refero** — SaaS/product UI, dashboards, tables, navigation, AI-интерфейсы. Особенно близко к Ekho. ([Refero Styles][3])
**Приоритет: 10/10.**
---
### 2. Визуальная насмотренность
4. **Godly** — сильные современные сайты, motion, необычные interaction ideas. ([godly.website][4])
5. **SiteInspire** — более спокойный, чистый и минималистичный веб. ([siteinspire.com][5])
6. **Minimal Gallery** — typography, spacing, hierarchy, minimal layouts. ([framify.design][6])
7. **Awwwards** — иногда слишком show-off, но отлично для необычных идей.
**Для Ekho:** SiteInspire/Minimal → база, Godly/Awwwards → специи.
---
### 3. Motion / те самые «приятные штуки»
8. **Emil Kowalski / animations.dev** — обязательно. Именно design engineering, micro-interactions, animations и polish. ([animations.dev][7])
9. **Motion.dev** — springs, layout transitions, gestures, hover/press, scroll, exit/enter animations. ([Motion][8])
10. **Codrops** — экспериментальные interactions, transitions, hover effects.
11. **GSAP demos/showcase** — сложные motion-примеры, когда реально понадобится.
Но motion должен давать **feedback или помогать понимать изменение состояния**, а не просто двигать всё подряд. ([Nielsen Norman Group][9])
---
### 4. Чтобы интерфейс был удобным, а не просто красивым
12. **Nielsen Norman Group** — UX-база. Особенно 10 usability heuristics. ([Nielsen Norman Group][10])
13. **Apple Human Interface Guidelines** — hierarchy, interaction, feedback, accessibility, controls. ([Apple Developer][11])
14. **Material Design 3** — изучать states, components, tokens и interaction rules, **не копировать визуальный стиль Google**. ([Material Design][12])
---
### 5. Продукты, которые надо буквально разбирать руками
Для Ekho я бы постоянно изучал:
* **Linear** — navigation, keyboard, speed, motion
* **Raycast** — command interfaces
* **Arc / Dia** — personality + interaction
* **Stripe** — information hierarchy
* **Notion** — progressive disclosure
* **Airbnb** — search/filter/complex information
* **Apple** — polish
* **Vercel** — developer-grade minimalism
Не «сделать как Linear», а спрашивать:
**Что происходит → почему это приятно → какую проблему решает → можно ли принцип перенести в Ekho.**
---
## Какие паттерны специально собирать
Создай себе **Ekho UI Library** и складывай туда:
`Navigation / Search / Command menu / University cards / Tables / Filters / Status / Progress / Tasks / Deadlines / Calendar / Forms / Empty states / Loading / Error / Toast / Hover / Selection / Drag & drop / Modals / Popovers / Tooltips / Onboarding / Animations`
И на каждый сохраняй **3–5 лучших референсов**.
### Главное правило
**80% знакомого + 20% фирменного Ekho.**
Пользователь не должен заново учиться пользоваться приложением. Знакомые conventions снижают когнитивную нагрузку; уникальность лучше создавать через детали, motion, typography, copy, transitions и несколько собственных interaction patterns. ([Nielsen Norman Group][13])
Если брать **только 5 ресурсов**, то: **Mobbin → Page Flows → Refero → Godly → animations.dev**. Это уже даст очень сильную базу.
[1]: https://mobbin.com/?utm_source=chatgpt.com "Mobbin — UI & UX design inspiration for mobile & web apps"
[2]: https://pageflows.com/?utm_source=chatgpt.com "UI/UX Design Inspiration & User Flows from Top Apps — Page ..."
[3]: https://styles.refero.design/examples/dashboard-ui-design?utm_source=chatgpt.com "Dashboard UI Design Examples"
[4]: https://godly.website/?utm_source=chatgpt.com "Recent — Design Inspiration"
[5]: https://www.siteinspire.com/?utm_source=chatgpt.com "Siteinspire: Best Website Design Inspiration"
[6]: https://framify.design/blog/website-inspiration-for-framer-sites?utm_source=chatgpt.com "11 Best Website Inspiration Sites for High-End Design (2026)"
[7]: https://animations.dev/skills?utm_source=chatgpt.com "Skills"
[8]: https://motion.dev/docs/animate?utm_source=chatgpt.com "animate() | Create JavaScript, SVG animations | Motion"
[9]: https://www.nngroup.com/articles/visibility-system-status/?utm_source=chatgpt.com "Visibility of System Status (Usability Heuristic #1)"
[10]: https://www.nngroup.com/articles/ten-usability-heuristics/?utm_source=chatgpt.com "10 Usability Heuristics for User Interface Design"
[11]: https://developer.apple.com/design/human-interface-guidelines?utm_source=chatgpt.com "Human Interface Guidelines - Design"
[12]: https://m3.material.io/foundations/design-tokens?utm_source=chatgpt.com "Design tokens – Material Design 3"
[13]: https://www.nngroup.com/articles/consistency-and-standards/?utm_source=chatgpt.com "Maintain Consistency and Adhere to Standards (Usability ..."
