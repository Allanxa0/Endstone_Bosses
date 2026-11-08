# Endstone Bosses

Plugin de bosses para servidores [Endstone](https://endstone.dev) (Minecraft Bedrock).

## Estado

Incluye el registro de bosses, atributos configurables, zonas de spawneo, aparición automática con enfriamiento, anuncio global y un sistema de combate dinámico (niveles, regeneración, minions, explosiones, rayos y efectos negativos). Incluye además recompensas por boss, clasificación Top 5 por daño y drops de minions, administrables desde el juego.

## Bosses

| Tipo | Entidad |
|------|---------|
| Pigman | `minecraft:zombie_pigman` |
| Piglin | `minecraft:piglin` |
| Blaze | `minecraft:blaze` |
| Bruga | `minecraft:piglin_brute` |
| Warden | `minecraft:warden` |
| Miniboss | `minecraft:silverfish` |

Endstone no permite generar zombis bebé, por eso el miniboss usa Silverfish.

## Comando

| Comando | Permiso | Por defecto |
|---------|---------|-------------|
| `/bosses` | `bosses.command.bosses` | Operadores |
| `/bossrank` | `bosses.command.info` | Todos |
| `/bossrewards` | `bosses.command.info` | Todos |

`/bossrank` muestra el ranking de daño en vivo del combate de boss más cercano (en la misma dimensión) junto con la recompensa configurada para cada puesto. `/bossrewards` abre un formulario de solo lectura con los drops, las recompensas del Top 5 y los drops de minions de cada boss.

`/bosses` abre un formulario donde se elige un boss para editar:

- **Atributos**: nombre personalizado (admite códigos `&`), multiplicador de vida, multiplicador de daño y enfriamiento, que se escriben como números (admite decimales; los valores fuera de rango se ajustan al límite).
- **Zonas de spawneo**: añadir la posición actual del jugador (con su dimensión) y eliminar zonas existentes.
- **Recompensas**: drops del boss, recompensas del Top 5 y drops de sus minions (ver [Recompensas](#recompensas)).

Los atributos se aplican solo a los bosses que se generen después y nunca a otras entidades.

## Aparición

Cada 10 segundos el plugin revisa cada tipo de boss. Si su enfriamiento terminó, aparece un boss en cada zona libre cuyo chunk esté cargado. Una zona nunca tiene dos bosses del mismo tipo vivos a la vez; otras zonas del mismo tipo siguen disponibles. El enfriamiento de un tipo comienza cuando muere uno de sus bosses y se conserva tras reiniciar.

Las dimensiones se guardan por nombre, por lo que funcionan también las personalizadas si existen en el servidor. Una zona cuya dimensión no existe se omite.

Endstone no permite modificar la velocidad de mobs, por eso no hay multiplicador de velocidad.

## Combate

El combate se activa en cuanto un jugador golpea a un boss y se desactiva por tipo con `combat.disabled_types` (por defecto el Miniboss queda fuera).

**Niveles.** El nivel depende de la vida restante del boss. `levels.thresholds` indica la fracción de vida a la que empieza cada nivel (por defecto 80 %, 60 %, 40 % y 20 %, es decir, niveles 1 a 5). Cada nivel suma `damage_bonus` al daño, reduce los cooldowns (`cooldown_scale`), aumenta las probabilidades (`chance_bonus`) y amplía el presupuesto de efectos negativos. Al subir de nivel se avisa a los jugadores cercanos y el nametag muestra nivel y vida.

**Regeneración.** Si el boss pasa `regeneration.delay` segundos sin recibir daño, recupera `regeneration.rate` de su vida máxima por segundo. Un golpe la interrumpe y baja el nivel conforme recupera vida.

**Inteligencia de combate.** El boss analiza una ventana deslizante de `combat.window` segundos con el daño recibido (como fracción de su vida máxima), los golpes, los atacantes distintos y los jugadores cercanos. Con esos datos puntúa cada habilidad, tira su probabilidad y elige una de las que superan la tirada con peso proporcional a su puntuación. Entre dos habilidades pasan al menos `combat.global_cooldown` segundos, así que nunca lanza varias a la vez.

| Habilidad | Se favorece cuando | Qué hace |
|-----------|--------------------|----------|
| `minions` | Recibe golpes (más si son rápidos, `rapid_hits`) | Invoca Silverfish con vida, daño y nombre configurables, hasta `max_alive` |
| `blast` | Al menos `attackers` jugadores distintos lo atacan | Coloca TNT en anillo a su alrededor; la mecha da tiempo de huir |
| `rage` | Pierde `burst` de su vida en la ventana | Sube su daño `damage_bonus` durante `duration` segundos |
| `lightning` | Hay jugadores cerca | Marca `strikes` posiciones y cae el rayo tras `delay` ticks |
| `curse` | Siempre disponible; más probable con mucha presión | Aplica efectos negativos a los `max_targets` jugadores que más daño hacen |

**Rayos.** El objetivo se elige por pesos: más probable para quien más daño hace y para quien está lejos (`kite_range`). Un jugador alcanzado no vuelve a ser objetivo durante `target_cooldown`. El rayo cae en la posición marcada, por lo que se puede esquivar.

**Efectos negativos.** Cada vez que un boss aplica efectos a un jugador, este recibe un mensaje en el chat con el nombre personalizado del boss y los efectos. Veneno, Wither, Levitación, Náuseas, Ceguera, Hambre, Lentitud y Debilidad se agrupan en cuatro familias (daño, movilidad, visión y sustento) y nunca se combinan dos de la misma familia. Cada efecto tiene un coste y cada aplicación un presupuesto (`effects.budget` más `budget_per_level` por nivel). La elección depende de la situación (varios atacantes, un único atacante o daño explosivo) y de la habilidad que los provoca (`lightning` y `rage` también pueden añadirlos con `rider_chance`). Tras aplicar efectos, el jugador queda protegido de nuevos efectos hasta que terminen más `effects.grace` segundos, y con menos de `effects.safe_health` de vida no recibe veneno ni Wither. La Levitación incluye Caída lenta para evitar daño por caída.

**Minions.** Se etiquetan con su boss, se eliminan al morir este (`remove_on_death`) y su vida y daño crecen `level_bonus` por nivel del boss. El daño de los Silverfish se aplica al golpear, sustituyendo al daño base.

**Limitaciones de la API.** Endstone no expone efectos de poción, explosiones ni rayos: los efectos se aplican con el comando `/effect` ejecutado desde la consola, y los rayos y el TNT se generan como entidades (`minecraft:lightning_bolt` y `minecraft:tnt`). El TNT tiene la mecha vanilla y su explosión no destruye bloques salvo que `abilities.blast.break_blocks` sea `true`. Tampoco permite modificar atributos de mobs, por eso el daño de los minions se fija al recibir el golpe.

## Recompensas

Cada tipo de boss tiene tres listas independientes, editables desde `/bosses` > boss > **Recompensas**:

| Lista | Cuándo se entrega |
|-------|-------------------|
| **Drops del boss** | Se sueltan en el suelo donde muere el boss |
| **Recompensas Top 5** | Una lista por puesto (1 a 5); se entregan al inventario del jugador de ese puesto (lo que no quepa cae a sus pies) |
| **Drops de minions** | Se sueltan donde muere cada Silverfish invocado por ese boss |

Cada objeto tiene cantidad mínima, cantidad máxima y probabilidad (0.01 % a 100 %, se escribe directamente y admite decimales como 0.1), y la tirada es independiente por objeto. Para añadir objetos:

- **Añadir objeto en mano**: usa el objeto que sostienes.
- **Añadir desde inventario**: lista tu inventario para elegir un objeto.

El objeto conserva su nombre, lore y encantamientos, y al añadirlo se abre el formulario de cantidad y probabilidad (por defecto, la cantidad del stack y 100 %). Pulsar un objeto de la lista permite editarlo o eliminarlo. Cantidades superiores al tamaño de stack se reparten en varios stacks.

**Top 5.** Cada golpe de un jugador a un boss suma su daño (limitado a la vida restante). Al morir el boss se anuncia en el chat su nombre y los cinco jugadores con más daño, con su puesto, daño y recompensa recibida. Con menos de cinco jugadores solo se listan los que participaron. Un jugador desconectado en ese momento aparece en la clasificación pero no recibe recompensa. Los mensajes se editan en `[messages]`.

**Solo bosses del plugin.** El daño, el ranking y las recompensas solo se activan en bosses generados por el sistema de spawneo (llevan una etiqueta propia) y los drops de minions solo en los Silverfish invocados por un boss. Fuera de las dimensiones con zonas de spawn, un Pigman, un Silverfish o cualquier otra entidad del mismo tipo que no sea del plugin no participa. Dentro de esas dimensiones, una entidad sin etiqueta del tipo de un boss que reciba un golpe de un jugador se adopta como boss y empieza a registrarse. Cada combate se lleva por separado, así que dos bosses del mismo tipo en zonas distintas no mezclan su clasificación. Los minions eliminados al morir su boss (`remove_on_death`) no sueltan drops.

Endstone no ofrece interfaces de contenedor, por eso los objetos se eligen con formularios en lugar de un cofre.

## Datos

`plugins/bosses/data.json` guarda por tipo el nombre, los multiplicadores, el enfriamiento, las zonas y el fin del enfriamiento actual. `plugins/bosses/rewards.json` guarda los drops, el Top 5 y los drops de minions de cada tipo. Ambos se editan desde `/bosses`. `plugins/bosses/state.json` guarda el estado de cada boss vivo (vida, daño acumulado por jugador, posición y zona) para que sobreviva a la descarga de chunks y a los reinicios; no hace falta editarlo. Si al volver a cargarse el chunk el boss ya no existe, se vuelve a generar en su última posición con su vida, vida máxima y ranking de daño guardados (hasta 20 segundos después de que un jugador se acerque a menos de 32 bloques).

## Configuración

`plugins/bosses/config.toml`:

```toml
spawn_broadcast = [
    "&a&lThe {bossName} BOSS has spawned!",
    "&eGo to X: {x} Y: {y} Z: {z} to kill the {bossName}!",
]
```

Cada línea se envía a todos los jugadores cuando aparece un boss. Usa `&` para colores. Variables: `{bossName}`, `{bossType}`, `{bossDamage}`, `{bossLife}`, `{dimension}`, `{x}`, `{y}`, `{z}`.

Los cambios de `config.toml` se aplican al reiniciar el servidor. Si falta una sección, se usan los valores por defecto.

| Sección | Claves |
|---------|--------|
| `[combat]` | `disabled_types`, `think_interval` (ticks, 60 por defecto), `window`, `global_cooldown`, `radius` |
| `[levels]` | `thresholds`, `damage_bonus`, `cooldown_scale`, `chance_bonus`, `name_format` (`{name}`, `{level}`, `{health}`, `{max}`) |
| `[regeneration]` | `enabled`, `delay`, `rate` |
| `[abilities.<habilidad>]` | `enabled`, `chance`, `cooldown` y las claves propias de cada habilidad |
| `[abilities.minions]` | `rapid_hits`, `min_count`, `max_count`, `max_alive`, `health`, `damage`, `level_bonus`, `show_name`, `name` (`{boss}`), `spawn_radius`, `remove_on_death` |
| `[abilities.blast]` | `attackers`, `count`, `radius`, `break_blocks` |
| `[abilities.lightning]` | `strikes`, `delay`, `target_cooldown`, `kite_range`, `impact_radius` |
| `[abilities.rage]` | `burst`, `duration`, `damage_bonus` |
| `[abilities.curse]` | `burst`, `max_targets` |
| `[effects]` | `duration_scale`, `amplifier_bonus`, `budget`, `budget_per_level`, `grace`, `safe_health`, `rider_chance` |
| `[effects.<efecto>]` | `enabled`, `seconds`, `amplifier` para `poison`, `wither`, `levitation`, `nausea`, `blindness`, `hunger`, `slowness` y `weakness` |
| `[messages]` | `level_up`, `rage`, `lightning`, `blast`, `curse` (`{boss}`, `{effects}`), `ranking_title` (`{boss}`), `ranking_line` (`{position}`, `{player}`, `{damage}`, `{rewards}`), `no_reward` |

El multiplicador de vida y de daño del boss, y su enfriamiento de aparición, siguen editándose desde `/bosses`.

## Instalación

```bash
pip install build
python -m build
```

Copia el `.whl` de `dist/` a la carpeta `plugins/` del servidor.

## Autor

Allanxa0
