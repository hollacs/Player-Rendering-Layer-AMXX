## Managing set_rendering in AMXX: A Layer System

Have you ever encountered difficulties managing multiple plugins that simultaneously use `set_rendering` in AMXX? 
I've made a layer system to deal with this issue.

### When Do set_rendering Conflicts Happen?

Imagine a zombie game mode where players can be frozen by an ice bomb. When frozen, the game applies a light blue glow effect using set_rendering. After the freeze wears off, the glow disappears.

Now, let’s say your zombies also have a berserk mode that adds a red glow to their bodies. This berserk effect is also managed using set_rendering. The problem arises when both effects happen at the same time:

If a player is in berserk mode and gets frozen, the blue glow vanishes as soon as the berserk mode ends (even if the freeze is still active).

Conversely, if a player is frozen while in berserk mode and the freeze ends before the berserk mode, the red glow might disappear too early. Ideally, it should stay until the berserk mode truly ends.

(Note: This is just an example; your actual game mode may handle these situations differently.)

Additionally, if you have special infected characters emitting a base green glow, managing all these effects can become tricky.

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