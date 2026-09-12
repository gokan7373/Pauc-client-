# Pauc-client-
import net.minecraft.client.MinecraftClient;
import net.minecraft.entity.player.PlayerEntity;
import net.minecraft.util.math.MathHelper;
import net.minecraft.util.math.Vec3d;

public class TargetUtils {

    // En yakın düşman oyuncuyu bulur
    public static PlayerEntity getClosestTarget(double range) {
        MinecraftClient mc = MinecraftClient.getInstance();
        if (mc.player == null || mc.world == null) return null;

        PlayerEntity closest = null;
        double minDistance = range * range;

        for (PlayerEntity player : mc.world.getPlayers()) {
            if (player == mc.player || !player.isAlive()) continue;

            double distSq = mc.player.squaredDistanceTo(player);
            if (distSq < minDistance) {
                minDistance = distSq;
                closest = player;
            }
        }
        return closest;
    }

    // Hedefe bakmak için gerekli Yaw ve Pitch açılarını hesaplar
    public static float[] getRotationsToEntity(PlayerEntity target) {
        MinecraftClient mc = MinecraftClient.getInstance();
        Vec3d eyePos = mc.player.getEyePos();
        Vec3d targetPos = target.getEyePos();

        double dx = targetPos.x - eyePos.x;
        double dy = targetPos.y - eyePos.y;
        double dz = targetPos.z - eyePos.z;

        double dist = Math.sqrt(dx * dx + dz * dz);
        float yaw = (float) Math.toDegrees(Math.atan2(dz, dx)) - 90.0F;
        float pitch = (float) -Math.toDegrees(Math.atan2(dy, dist));

        return new float[]{
            MathHelper.wrapDegrees(yaw),
            MathHelper.wrapDegrees(pitch)
        };
    }
}
public class ElytraHelper {

    public static void updateElytraTargeting() {
        MinecraftClient mc = MinecraftClient.getInstance();
        if (mc.player == null || !mc.player.isFallFlying()) return;

        PlayerEntity target = TargetUtils.getClosestTarget(50.0); // 50 blok menzil
        if (target != null) {
            float[] rotations = TargetUtils.getRotationsToEntity(target);
            
            // Oyuncunun bakış yönünü hedefe kilitler
            mc.player.setYaw(rotations[0]);
            mc.player.setPitch(rotations[1]);
        }
    }
}import net.minecraft.util.Hand;

public class KillAura {

    public static void onTick() {
        MinecraftClient mc = MinecraftClient.getInstance();
        if (mc.player == null || mc.interactionManager == null) return;

        PlayerEntity target = TargetUtils.getClosestTarget(4.0); // 4 blok saldırı menzili
        if (target != null) {
            // Saldırı bekleme süresi dolduysa vur
            if (mc.player.getAttackCooldownProgress(0.5f) >= 1.0f) {
                float[] rotations = TargetUtils.getRotationsToEntity(target);
                
                // Bakış açısını hedefe yönlendir
                mc.player.setYaw(rotations[0]);
                mc.player.setPitch(rotations[1]);

                // Vuruş yap
                mc.interactionManager.attackEntity(mc.player, target);
                mc.player.swingHand(Hand.MAIN_HAND);
            }
        }
    }
}import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.event.lifecycle.v1.ClientTickEvents;

public class PeucClientMod implements ClientModInitializer {

    @Override
    public void onInitializeClient() {
        ClientTickEvents.END_CLIENT_TICK.register(client -> {
            if (client.player != null) {
                KillAura.onTick();
                ElytraHelper.updateElytraTargeting();
            }
        });
    }
}


