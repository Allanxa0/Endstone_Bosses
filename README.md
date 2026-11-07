# Endstone Bosses

Plugin de bosses para servidores [Endstone](https://endstone.dev) (Minecraft Bedrock).

## Estado

Incluye el registro de bosses, atributos configurables, zonas de spawneo, aparición automática con enfriamiento y anuncio global. Las recompensas y otras mecánicas llegarán en futuras versiones.

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

## Instalación

```bash
pip install build
python -m build
```

Copia el `.whl` de `dist/` a la carpeta `plugins/` del servidor.

## Autor

Allanxa0
