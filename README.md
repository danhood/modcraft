# Minecraft Modding 101

Instructions for a lab on Minecraft Modding

## Setup

1. Let's open a shell on Windows, Start → Search and type `powershell`
2. This will open a blue console that we're going to issue commands in
3. Let's first type `cd` a space, and then let's drag the folder called `modding` at the top right of the Desktop and
   drop in the shell - this will input text that's the location of that folder, then press `Enter`
4. Type `dir` to see what's here, then `cd forge-1.19-41.1.0-mdk` to enter the forge directory
5. Run the following command, be sure that the slash is in the right direction with no extra spaces, this will take a
   while and we'll come back to it

```
.\gradlew genEclipseRuns
```

6. While that's running, let's open the Eclipse Integrated Development Environment (IDE) by opening the `modding`
   folder, clicking on `eclipse-java-2023-03-R-win32-x86_64` → `eclipse` → `eclipse`
7. Go back to the blue console, we're waiting for the command we ran to say `BUILD SUCCESSFUL`
8. Once that's done, go back to Eclipse, chose menu `File` → `Import…` → select `Gradle` → `Existing Gradle Project`,
   click the `Next >` button two times, choose the location of forge directory, and then click `Finish`
9. Double click on `forge` folder in `Gradle Task` on the right side, double click on `forgegradle runs`, and select
   `runClient`
10. If all goes well, we'll have built any mods we have and launch Minecraft with them installed

## Give Me All the Things

Create a file called `ChatItems.java` in `com.example.examplemod` with the following contents:

```java
package com.example.examplemod;

import net.minecraft.world.entity.player.Inventory;
import net.minecraft.world.item.ItemStack;
import net.minecraft.world.item.Items;
import net.minecraftforge.event.ServerChatEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

@Mod.EventBusSubscriber(modid = ExampleMod.MODID)
public class ChatItems {

    // listen to the chat
    @SubscribeEvent
    public static void giveItems(ServerChatEvent event) {

        // get the message as text
        String message = event.getMessage();

        // get the play's inventory
        Inventory inventory = event.getPlayer().getInventory();

        // if message contains 'potato', give us potatoes
        if (message.contains("potato")) {
            inventory.add(new ItemStack(Items.POTATO, 64));
        }

        // what more can we do?
    }

}
```

Save the file, then we can re-run our build by simply pressing the green play button

Press `T` to enter the chat and type something that contains the word "potato" and press `Enter`.

What else we can give ourselves via chat?

## Creeper Has Entered the Chat

Create a file called `CreeperSpawnAlert.java` in `com.example.examplemod` with the following contents:

<!-- NOTE: below is intentionally missing a Creeper import, explain imports -->

```java
package com.example.examplemod;

import net.minecraft.ChatFormatting;
import net.minecraft.network.chat.Component;
import net.minecraft.world.entity.player.Player;
import net.minecraftforge.event.entity.EntityJoinLevelEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

@Mod.EventBusSubscriber(modid = ExampleMod.MODID)
public class CreeperSpawnAlert {

	@SubscribeEvent
	public static void sendAlert(EntityJoinLevelEvent event) {

		if (event.getEntity() instanceof Creeper && event.getLevel().isClientSide) {
			for (Player player : event.getLevel().players()) {
				player.sendSystemMessage(Component.literal(ChatFormatting.GREEN + "A creeper has spawned!"));
			}
		}
	}

}
```

Save and re-run build by pressing the green play button

Change to night time by typing `/time set night` and press `Enter`.

How do we display text in other colors?

## Uninvited Guests

Create a file called `DragonSpawner.java` in `com.example.examplemod` with the following contents:

```java
package com.example.examplemod;

import net.minecraft.world.entity.EntityType;
import net.minecraft.world.entity.boss.enderdragon.EnderDragon;
import net.minecraft.world.entity.boss.enderdragon.phases.EnderDragonPhase;
import net.minecraft.world.level.block.Blocks;
import net.minecraftforge.event.level.BlockEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

@Mod.EventBusSubscriber(modid = ExampleMod.MODID)
public class DragonSpawner {

	@SubscribeEvent
	public static void spawnDragon(BlockEvent.EntityPlaceEvent event) {

		if (event.getPlacedBlock().getBlock() == Blocks.DRAGON_EGG) {
			event.getLevel().removeBlock(event.getPos(), false); // false = no flags
			EnderDragon dragon = EntityType.ENDER_DRAGON.create(event.getEntity().getLevel());
			dragon.moveTo(event.getPos(), 0, 0);
			dragon.getPhaseManager().setPhase(EnderDragonPhase.TAKEOFF);
			event.getLevel().addFreshEntity(dragon);
		}

	}

}
```

Save and re-run build by pressing the green play button

Spawn above the terrain:

```java
dragon.moveTo(event.getPos().above(2), 0, 0);
```

## You're Gonna Poke Someone's Eye Out

Create a file called `SharpSnowballs.java` in `com.example.examplemod` with the following contents:

```java
package com.example.examplemod;

import net.minecraft.world.entity.Entity;
import net.minecraft.world.entity.EntityType;
import net.minecraft.world.entity.projectile.Arrow;
import net.minecraft.world.entity.projectile.Snowball;
import net.minecraft.world.level.Level;
import net.minecraftforge.event.entity.EntityJoinLevelEvent;
import net.minecraftforge.eventbus.api.SubscribeEvent;
import net.minecraftforge.fml.common.Mod;

@Mod.EventBusSubscriber(modid = ExampleMod.MODID)
public class SharpSnowballs {
    @SubscribeEvent
    public static void replaceSnowballWithArrow(EntityJoinLevelEvent event) {
        Entity snowball = event.getEntity();
        Level level = event.getLevel();

        if (!(snowball instanceof Snowball)) {
            return;
        }

        if (!level.isClientSide) {
            Arrow arrow = EntityType.ARROW.create(level);
            arrow.moveTo(snowball.position());
            arrow.setDeltaMovement(snowball.getDeltaMovement());
            level.addFreshEntity(arrow);
        }

        event.setCanceled(true);
    }
}
```

Save and re-run build by pressing the green play button

Press `E` to bring inventory and assign Snowballs to an inventory slot.

## Boom Goes the Dynamite

What if we replace arrows with TNT?

```java
PrimedTnt tnt = EntityType.TNT.create(level);
tnt.setFuse(80);
```

## Thanks

- Thanks to the [CNCF's Kid’s Day Minecraft Modding Post][cncf] for inspiring us to do this at our company's Take Your
  Daughters and Sons to Work Day
- An unbelievable amount of thanks goes out to [Devoxx4Kids][d4k] for their incredible [Minecraft Modding using
  Forge][d4k-modding] guide which this is borrowed from

## Resources

This lab uses specifically the following versions of downloads to alleviate any compatibility issues:

- [JDK 17.0.7](https://download.oracle.com/java/17/archive/jdk-17.0.7_windows-x64_bin.msi)
- [Forge 1.19-41.1.0](https://maven.minecraftforge.net/net/minecraftforge/forge/1.19-41.1.0/forge-1.19-41.1.0-mdk.zip)
- [Eclipse IDE for Java Developers 2023-03-R-win32-x86_64](https://www.eclipse.org/downloads/download.php?file=/technology/epp/downloads/release/2023-03/R/eclipse-java-2023-03-R-win32-x86_64.zip)

[cncf]:
  https://www.cncf.io/blog/2023/03/22/cloud-native-youth-encouraging-the-next-generation-of-technologies-with-kids-day/
[d4k]: https://www.devoxx4kids.org/
[d4k-modding]: https://github.com/devoxx4kids/materials/blob/master/workshops/minecraft/readme-forge.asciidoc
