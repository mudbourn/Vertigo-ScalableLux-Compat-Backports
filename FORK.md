# Vertigo — ScalableLux compat backport (MMS fork)

Fork of [Builderb0y/Vertigo](https://github.com/Builderb0y/Vertigo), branched from
`Pre-26.1` at `mod_version = 1.2.4` (the 1.21.11 line). MIT, same as upstream —
see `LICENSE.txt`.

Built and shipped as **`1.2.4+mms.1`**.

## Why this fork exists

On 1.21.11, stock Vertigo 1.2.4 and ScalableLux are mutually incompatible: loading
both crashes. The cause is a single wrong callback type.

`ScalableLux_ChunkAccessMixin_Undoing#vertigo_dontSkipInit` is a MixinSquared
handler injected at `HEAD` of starlight's `ChunkAccessMixin#skipInit`. That target
returns `boolean`, so the handler must take `CallbackInfoReturnable<Boolean>`.
Upstream declares it as `CallbackInfo`, and mixin refuses the injection.

## The entire patch

`src/common/java/builderb0y/vertigo/mixin/ScalableLux_ChunkAccessMixin_Undoing.java`:

```diff
-import org.spongepowered.asm.mixin.injection.callback.CallbackInfo;
+import org.spongepowered.asm.mixin.injection.callback.CallbackInfoReturnable;

-	private void vertigo_dontSkipInit(ChunkSkyLight skyLight, Chunk chunk, CallbackInfo callback) {
+	private void vertigo_dontSkipInit(ChunkSkyLight skyLight, Chunk chunk, CallbackInfoReturnable<Boolean> callback) {
```

Plus `mod_version = 1.2.4+mms.1` in `gradle_common.properties`. Nothing else
differs from upstream.

## Still needed?

Yes as of upstream `V1.2.6` — the `CallbackInfo` signature is unchanged there, so
this is not fixed upstream and the fork can't simply be dropped for a newer
release. Re-check that one line before rebasing onto any future tag; if upstream
corrects it, retire this repo and point the pack back at stock Vertigo.

## Building

Upstream's multi-version setup: switch to the target MC version, then build.

```bash
./gradlew "Switch to 1.21.11" && ./gradlew build
```

Output lands in `build/libs/Vertigo-1.2.4+mms.1-MC1.21.11.jar`. `buildAll.sh`
hardcodes upstream's own `JAVA_HOME`; set your own before using it.

## Consumed by

`mms-pack` indexes this jar as `mods/vertigo.pw.toml`, `side = "client"`, pinned.
The pack needs a real download URL, so every rebuild has to be published as a
release asset here before the pack can point at it.
