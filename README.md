## Managing set_rendering in AMXX: A Conflict Resolution System

Have you ever encountered difficulties managing multiple plugins that simultaneously use `set_rendering` in AMXX? 
I've made a managing system to deal with this issue.

### When Does set_rendering Conflict Occur?

Let's consider an example:

In a zombie mode, there's an ice bomb that freezes players. When frozen, the game applies a light blue rendering effect using `set_rendering`. After the freeze is lifted, the rendering is reset.

Now, imagine your zombies have a berserk mode that adds a red glow to their bodies. When activated, this effect is also managed using `set_rendering`. However, conflicts arise when both effects occur simultaneously.

For instance:
1. If a player is in berserk mode and gets frozen, the blue effect will disappear as soon as the berserk mode ends (even if the freeze still remains)
2. Conversely, if a player is frozen while in berserk mode and the freeze is lifted before the berserk mode completes, the red glow effect might vanish too early. Ideally, it should persist until the berserk mode truly ends.

(Note that this is just an illustrative example; your actual implementation may handle these scenarios differently.)

Additionally, if you have special infected characters emitting a base green glow, encountering the situations described above can become quite problematic.

## Usage Instructions:

```sourcepawn
// Add a new rendering layer
// The first 5 parameters are similar to set_rendering
// duration specifies the glow's duration (set to 0.0 for permanent until next respawn)
// class provides a name for the layer; if a layer with the same name exists, its rendering style is updated
// zindex determines the layer's priority (higher values take precedence)
// Returns the inserted index
native render_push(id, fx=kRenderFxNone, color[3]={0, 0, 0}, mode=kRenderNormal, amount=0, Float:duration=0.0, const class[]="", zindex=0);

// Remove a specific rendering layer
// id is the player's ID
// pop_index specifies the index to remove (if -1, uses class to find the layer)
// class allows finding the layer by name (if empty, uses pop_index)
native render_pop(id, pop_index=-1, const class[]="");

// Remove all layers
native render_clear(id);

// Get the current rendering layer ID for a player
native render_current(id);

// Retrieve data for a specific layer (see render_layer_const.inc for details)
native render_get_data(id, index, data[RenderingData]);
```

### Notes:
- This API automatically removes all layers when a player respawns.
- By default, there's a limit of 16 layers; modify `MAX_LAYERS` in your .sma file to increase this limit.

## Example Scenarios Based on the Situations Mentioned Above:

1. Setting up the base green glow for special infected characters:

When a player becomes a special infected, add the following code: (zindex is 0)

```sourcepawn
render_push(id, kRenderFxGlowShell, {0, 255, 0}, kRenderNormal, 16, 0.0, "base", 0);
```

2. Setting up the blue glow when a zombie hit by a ice bomb: 

Assuming the freeze duration is 3 seconds, use:

```sourcepawn
render_push(id, kRenderFxGlowShell, {0, 200, 200}, kRenderNormal, 16, 3.0, "freeze", 10);
```

3. Setting up the red glow when the zombie is in berserk mode: 

Assuming the berserk mode lasts 10 seconds, add: (zindex is 1)

```sourcepawn
render_push(id, kRenderFxGlowShell, {255, 0, 0}, kRenderNormal, 16, 10.0, "berserk", 1);
```

4. To manually cancel the berserk effect in certain situations:

```sourcepawn
render_pop(id, -1, "berserk");
```

Feel free to adapt these examples to your specific game mode and plugin needs!