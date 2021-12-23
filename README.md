## Features

This plugin is primarily intended to be used by developers of other plugins.

- Allows other plugins to resize drones
- Allows privileged players to resize drones with a command
- Allows attaching entities to resized drones without resizing those attachments

## Required plugins

- [Entity Scale Manager](https://umod.org/plugins/entity-scale-manager) -- No configuration or permissions needed.

## Permissions

- `dronescalemanager.unrestricted` -- Allows unrestricted usage of the `scaledrone` command.

## Commands

- `scaledrone <size>` -- Resizes the drone you are looking at.
  - This works like `scale <size>` from [Entity Scale Manager](https://umod.org/plugins/entity-scale-manager) except that it handles special needs of drones.

## Localization

```json
{
  "Error.NoPermission": "You don't have permission to do that.",
  "Error.Syntax": "Syntax: {0} <size>",
  "Error.NoDroneFound": "Error: No drone found.",
  "Scale.Success": "Drone was scaled to: {0}",
  "Error.ScalePrevented": "An error occurred while attempting to resize that drone."
}
```

## Developer API

When integrating with this plugin, the most important thing to understand is that a resized drone will actually be parented to a root sphere entity that has a 1.0 scale. This root entity is important for several reasons, but here is what you need to know.

- If you want to attach an entity to the drone without resizing the entity, attach it to the root entity using `API_ParentEntity(drone, entity)`.
- If you have a reference to an entity (such as from a hook call), and you want to check if it is parented to a resized drone, call the `API_GetParentDrone(entity)` method. This will return either the drone or `null` if the entity is not parented to the root entity of a drone.

#### API_ScaleDrone

```csharp
bool API_ScaleDrone(Drone drone, float scale)
```

- Resizes a drone.
- The max recommended size is `7.0`.

#### API_ParentEntity

```csharp
bool API_ParentEntity(Drone drone, BaseEntity entity)
```

- Parents the specified entity to the root entity of the resized drone, without resizing the entity
- The `localPosition` of the entity's transform will automatically be scaled according to the drone size
- Spawns the entity if not already spawned
- Returns `true` if the entity was parented successfully
- Returns `false` if the drone is a delivery drone or is not resized

#### API_ParentTransform

```csharp
bool API_ParentTransform(Drone drone, Transform childTransform)
```

- Parents the specified transform to the root entity
- The `localPosition` of the transform will automatically be scaled according to the drone size
- Returns `true` if the transform was parented successfully
- Returns `false` if the drone is a delivery drone or is not resized

#### API_GetParentDrone

```csharp
Drone API_GetParentDrone(BaseEntity entity)
```

- Returns the resized drone, given the root entity or a child of the root entity
- Returns `null` if the entity is not a root entity or is not parented to a resized drone's root entity
- If you expect to call this often, you can optimize performance by only calling after verifying that the entity has a parent

#### API_GetRootEntity

```csharp
BaseEntity API_GetRootEntity(Drone drone)
```

- Returns the root entity of a resized drone
- Returns `null` if the drone is not resized

## Developer Hooks

#### OnDroneScale

```csharp
bool? OnDroneScale(Drone drone, float scale)
```

- Called when a drone is about to be resized
- Returning `false` will prevent the drone from being resized
- Returning `null` will result in the default behavior

#### OnDroneScaleBegin

```csharp
void OnDroneScaleBegin(Drone drone, BaseEntity rootEntity, float scale, float previousScale)
```

- Called when a drone is about to be resized, after ensuring the drone has a root entity
- No return behavior
- Useful for plugins that need to move entities or game objects from the drone to the root entity before the resize actually happens

#### OnDroneScaled

```csharp
void OnDroneScaled(Drone drone, BaseEntity rootEntity, float scale, float previousScale)
```

- Called after a drone has been resized
- No return behavior
