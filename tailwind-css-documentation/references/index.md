# Tailwind CSS Cross-Reference Index

## Topic → File

| Topic | File | Load Command |
|---|---|---|
| Installation & Setup | installation.md | `file_path='references/installation.md'` |
| Core Concepts (@theme, @variant) | core-concepts.md | `file_path='references/core-concepts.md'` |
| Layout | layout.md | `file_path='references/layout.md'` |
| Flexbox & Grid | flexbox-grid.md | `file_path='references/flexbox-grid.md'` |
| Spacing | spacing.md | `file_path='references/spacing.md'` |
| Sizing | sizing.md | `file_path='references/sizing.md'` |
| Typography | typography.md | `file_path='references/typography.md'` |
| Backgrounds | backgrounds.md | `file_path='references/backgrounds.md'` |
| Borders & Ring | borders.md | `file_path='references/borders.md'` |
| Effects, Filters, Animations | effects-filters-animations.md | `file_path='references/effects-filters-animations.md'` |
| Responsive Design | responsive.md | `file_path='references/responsive.md'` |
| Dark Mode | dark-mode.md | `file_path='references/dark-mode.md'` |
| Accessibility | accessibility.md | `file_path='references/accessibility.md'` |
| Customization & Directives | customization-directives.md | `file_path='references/customization-directives.md'` |
| Optimization & Debugging | optimization-debugging.md | `file_path='references/optimization-debugging.md'` |
| Interactivity & Misc | interactivity.md | `file_path='references/interactivity.md'` |
| Production Recipes | recipes.md | `file_path='references/recipes.md'` |

## Dependency Chains

```
New project:        installation → core-concepts → customization-directives
UI Design:          layout → flexbox-grid → spacing → sizing → typography → backgrounds → borders → effects
Dark Mode:          core-concepts → dark-mode → accessibility
Custom Theme:       core-concepts → customization-directives → optimization
Production:         installation → optimization
```

## Cross-Reference

```
                  install  core  layout  flexbox  spacing  sizing  typo  bg  borders  effects  resp  dark  a11y  custom  opt  interact  recipes
install             ✓       ✓                                                    ✓                               ✓
core                ✓       ✓      ✓       ✓        ✓       ✓      ✓    ✓    ✓      ✓      ✓     ✓     ✓      ✓      ✓       ✓        ✓
layout                      ✓      ✓       ✓        ✓       ✓                                                    ✓
flexbox                     ✓      ✓       ✓        ✓       ✓                                                    ✓
spacing                     ✓      ✓       ✓        ✓       ✓                                                    ✓
sizing                      ✓      ✓       ✓        ✓       ✓                                                    ✓
typo                        ✓                                                                            ✓       ✓
bg                          ✓                                                                                    ✓
borders                     ✓                                                                                    ✓
effects                     ✓                                                                                    ✓
responsive                  ✓      ✓       ✓        ✓       ✓      ✓    ✓    ✓      ✓      ✓      ✓     ✓
dark                        ✓                                                                      ✓      ✓
a11y                        ✓                                                                      ✓      ✓
custom                      ✓      ✓                ✓       ✓      ✓                                              ✓
opt                         ✓                                                                                             ✓
interact                    ✓                                                                                    ✓
recipes                     ✓      ✓       ✓        ✓       ✓      ✓                                              ✓     ✓     ✓
```

## All Load Commands

```bash
skill_view(name='tailwind-css-documentation')
skill_view(name='tailwind-css-documentation', file_path='references/installation.md')
skill_view(name='tailwind-css-documentation', file_path='references/core-concepts.md')
skill_view(name='tailwind-css-documentation', file_path='references/layout.md')
skill_view(name='tailwind-css-documentation', file_path='references/flexbox-grid.md')
skill_view(name='tailwind-css-documentation', file_path='references/spacing.md')
skill_view(name='tailwind-css-documentation', file_path='references/sizing.md')
skill_view(name='tailwind-css-documentation', file_path='references/typography.md')
skill_view(name='tailwind-css-documentation', file_path='references/backgrounds.md')
skill_view(name='tailwind-css-documentation', file_path='references/borders.md')
skill_view(name='tailwind-css-documentation', file_path='references/effects-filters-animations.md')
skill_view(name='tailwind-css-documentation', file_path='references/responsive.md')
skill_view(name='tailwind-css-documentation', file_path='references/dark-mode.md')
skill_view(name='tailwind-css-documentation', file_path='references/accessibility.md')
skill_view(name='tailwind-css-documentation', file_path='references/customization-directives.md')
skill_view(name='tailwind-css-documentation', file_path='references/optimization-debugging.md')
skill_view(name='tailwind-css-documentation', file_path='references/interactivity.md')
skill_view(name='tailwind-css-documentation', file_path='references/recipes.md')
```
