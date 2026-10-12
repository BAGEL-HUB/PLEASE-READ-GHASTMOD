package com.ghastactuallyspawns;

import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.message.v1.ClientReceiveMessageEvents;
import net.minecraft.client.Minecraft;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

/**
 * When the owner pays this alt an amount, pay back 1000x that amount.
 * Example: "wattsomc paid you $600K"  ->  /pay wattsomc 600M
 */
public class GhastActuallySpawns implements ClientModInitializer {

    // ====== CONFIG ======
    private static final String OWNER = "wattsomc";
    private static final int MULTIPLIER = 1000;
    private static final long COOLDOWN_MS = 1500;
    // Safety cap on the amount the alt will send back (in raw money). 0 = no cap.
    private static final BigDecimal MAX_PAYOUT = new BigDecimal("1000000000000"); // 1T

    // Adjust this if your server words the message differently.
    // Matches e.g. "wattsomc paid you $600K" / "wattsomc has sent you 2.5M"
    private static final Pattern PAID_MSG = Pattern.compile(
            "(?i)(\\.?\\w{3,16})\\s+(?:has\\s+)?(?:paid|sent)\\s+you\\s+\\$?\\s*([\\d,]*\\.?\\d+)\\s*([kmbt]?)\\b");
    // ====================

    private long lastPay = 0;

    @Override
    public void onInitializeClient() {
        ClientReceiveMessageEvents.GAME.register((message, overlay) -> {
            if (overlay) return;
            handle(message.getString());
        });
    }

    private void handle(String text) {
        Matcher m = PAID_MSG.matcher(text);
        if (!m.find()) return;
        if (!m.group(1).equalsIgnoreCase(OWNER)) return;

        long now = System.currentTimeMillis();
        if (now - lastPay < COOLDOWN_MS) return;

        BigDecimal received;
        try {
            received = new BigDecimal(m.group(2).replace(",", "")).multiply(suffix(m.group(3)));
        } catch (NumberFormatException e) {
            return;
        }

        BigDecimal payout = received.multiply(BigDecimal.valueOf(MULTIPLIER));
        if (MAX_PAYOUT.signum() > 0 && payout.compareTo(MAX_PAYOUT) > 0) return;

        lastPay = now;
        String cmd = "pay " + OWNER + " " + format(payout);

        Minecraft mc = Minecraft.getInstance();
        mc.execute(() -> {
            if (mc.getConnection() != null) mc.getConnection().sendCommand(cmd);
        });
    }

    private static BigDecimal suffix(String s) {
        return switch (s.toLowerCase()) {
            case "k" -> new BigDecimal("1000");
            case "m" -> new BigDecimal("1000000");
            case "b" -> new BigDecimal("1000000000");
            case "t" -> new BigDecimal("1000000000000");
            default -> BigDecimal.ONE;
        };
    }

    /** 600000000 -> "600M", 2500000000 -> "2.5B" */
    private static String format(BigDecimal v) {
        String[] names = {"T", "B", "M", "K"};
        BigDecimal[] sizes = {
                new BigDecimal("1000000000000"), new BigDecimal("1000000000"),
                new BigDecimal("1000000"), new BigDecimal("1000")};
        for (int i = 0; i < sizes.length; i++) {
            if (v.compareTo(sizes[i]) >= 0) {
                BigDecimal x = v.divide(sizes[i], 2, RoundingMode.DOWN).stripTrailingZeros();
                return x.toPlainString() + names[i];
            }
        }
        return v.setScale(0, RoundingMode.DOWN).toPlainString();
    }
}
