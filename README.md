# Endstone Bosses

Plugin de bosses para servidores [Endstone](https://endstone.dev) (Minecraft Bedrock).

## Estado

Incluye el registro de bosses, atributos configurables, zonas de spawneo, aparición automática con enfriamiento, anuncio global y un sistema de combate dinámico (niveles, regeneración, minions, explosiones, rayos y efectos negativos). Las recompensas llegarán en futuras versiones.

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

Abre un formulario donde se elige un boss para editar:

- **Atributos**: nombre personalizado (admite códigos `&`), multiplicador de vida, multiplicador de daño y enfriamiento.
- **Zonas de spawneo**: añadir la posición actual del jugador (con su dimensión) y eliminar zonas existentes.

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

**Efectos negativos.** Veneno, Wither, Levitación, Náuseas, Ceguera, Hambre, Lentitud y Debilidad se agrupan en cuatro familias (daño, movilidad, visión y sustento) y nunca se combinan dos de la misma familia. Cada efecto tiene un coste y cada aplicación un presupuesto (`effects.budget` más `budget_per_level` por nivel). La elección depende de la situación (varios atacantes, un único atacante o daño explosivo) y de la habilidad que los provoca (`lightning` y `rage` también pueden añadirlos con `rider_chance`). Tras aplicar efectos, el jugador queda protegido de nuevos efectos hasta que terminen más `effects.grace` segundos, y con menos de `effects.safe_health` de vida no recibe veneno ni Wither. La Levitación incluye Caída lenta para evitar daño por caída.

**Minions.** Se etiquetan con su boss, se eliminan al morir este (`remove_on_death`) y su vida y daño crecen `level_bonus` por nivel del boss. El daño de los Silverfish se aplica al golpear, sustituyendo al daño base.

**Limitaciones de la API.** Endstone no expone efectos de poción, explosiones ni rayos: los efectos se aplican con el comando `/effect` ejecutado desde la consola, y los rayos y el TNT se generan como entidades (`minecraft:lightning_bolt` y `minecraft:tnt`). El TNT tiene la mecha vanilla y su explosión no destruye bloques salvo que `abilities.blast.break_blocks` sea `true`. Tampoco permite modificar atributos de mobs, por eso el daño de los minions se fija al recibir el golpe.

## Datos

`plugins/bosses/data.json` guarda por tipo el nombre, los multiplicadores, el enfriamiento, las zonas y el fin del enfriamiento actual. Se edita desde `/bosses`.

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
| `[combat]` | `disabled_types`, `think_interval` (ticks), `window`, `global_cooldown`, `radius` |
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
| `[messages]` | `level_up`, `rage`, `lightning`, `blast`, `curse` |

El multiplicador de vida y de daño del boss, y su enfriamiento de aparición, siguen editándose desde `/bosses`.

## Instalación

```bash
pip install build
python -m build
```

Copia el `.whl` de `dist/` a la carpeta `plugins/` del servidor.

## Autor

Allanxa0
