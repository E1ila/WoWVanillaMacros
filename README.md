# WoW Vanilla Macros

Most macros require [SuperMacro](https://github.com/Monteo/SuperMacro) addon, it adds some extra functions and allows writing more than 255 character macros.

If a macro isn't working, double check that you copied it correctly and got all prerequisites. If all looks well, open an issue, try to write as much defail as possible.

## Generic

#### Use Healthstone > Root Tuber (Supermacro)
When you have a Hearthstone & Whipper Root Tuber in your bags, you want to use first HS and only if got none, use root tuber. They share the same cooldown.
This function (put on the extended box) will find an item and use it in given order -
```
function FindAndUse(...)
    for i,v in ipairs(arg) do
        local bag,slot=FindItem(v)
        if bag and slot then
            UseContainerItem(bag,slot)
            return true
        end
    end
    return false
end
```
```
/run FindAndUse("Major Healthstone", "Greater Healthstone", "Whipper Root Tuber")
```

#### Get Spell ID by Name (Supermacro)
Some macro commands like `GetSpellCooldown` require the ID of spell you want to use, this is how you get the spell ID -
```
function GetSpellId(spellname, spellrank)
  local name,rank,i; 
  for i=1,200,1 do 
    name,rank = GetSpellName(i,BOOKTYPE_SPELL)
	if name and name == spellname and (not spellrank or spellrank == rank) then 
	  return i
	end
  end
  return nil
end
```
Or you can print all spells in your book (doesn't require supermacro) -
```
/script local name,rank,i; for i=1,200,1 do name,rank=GetSpellName(i,BOOKTYPE_SPELL); if name then DEFAULT_CHAT_FRAME:AddMessage(i .. " = " .. name .. " / " .. rank); end; end
```

#### Use Item in Bags (Supermacro)
Finds and uses an item in your bags by name -
```
function UseItem(item) 
  local bag,slot=FindItem(item)
  if (bag==nil or slot==nil) then 
    Printd("No more " .. item .. "!")
  else 
    UseContainerItem(bag,slot)
  end
end
```

## Healing

#### Heal Hovered Target
Macro to try casting a heal on one of these: friendly target being hovered by the mouse > current selected target > self -
```
/run local c=CastSpellByName s="Flash Heal" if UnitExists("mouseover") and UnitReaction("mouseover","player")>4 then TargetUnit("mouseover") c(s) TargetLastTarget() else c(s) end
```
Same macro as before, only as a function for easy use with Supermacro -
```
function CastHover(spellname) 
  if UnitExists("mouseover") and UnitReaction("mouseover","player")>4 then 
    TargetUnit("mouseover") 
	CastSpellByName(spellname) 
	TargetLastTarget() 
  else 
  	CastSpellByName(spellname) 
  end
end
```
```
/run CastHover("Flash Heal")
```

#### Shaman Emergency Healing
Will stop casting (won't work if global cooldown not finished), cast `Nature's Swiftness` for instant cast and cast `Healing Wave` on hovered target > selected target > self (one of which).
```
/run local c=CastSpellByName s="Healing Wave" SpellStopCasting() c("Nature's Swiftness") if UnitExists("mouseover") and UnitReaction("mouseover","player")>4 then TargetUnit("mouseover") c(s) TargetLastTarget() else c(s) end
```
or with Supermacro, don't forget to paste the function `CastHover` into the extended box (right side) -
```
/run SpellStopCasting()
/cast Nature's Swiftness
/run CastHover("Healing Wave")
```

## Rogue

#### Drop Aggro (Supermacro)
Will cast Vanish, if on cooldown will use Limited Invulnerability Potion. Requires [GetSpellId](#get-spell-id-by-name-supermacro) and [UseItem](#use-item-in-bags-supermacro) functions.
```
/run local VANISH_ID=GetSpellId("Vanish") if GetSpellCooldown(VANISH_ID, BOOKTYPE_SPELL)==0 then CastSpellByName("Vanish") else UseItem("Limited Invulnerability Potion") end
```

#### Coat Weapons with Poison
Set the name of poison you want to use, make sure you're idle when casting this, not mounted or anything. First click will coat left weapon, 2nd click the right weapon -
```
/run local bag,slot=FindItem("Instant Poison VI") if slot then if not tmpPoisonSide then UseContainerItem(bag,slot) PickupInventoryItem(16) ReplaceEnchant() ClearCursor() tmpPoisonSide=1 else UseContainerItem(bag,slot) PickupInventoryItem(17) ReplaceEnchant() ClearCursor() tmpPoisonSide=nil end else print("OUT OF POISON!!!") end
```

## Mage

#### Mana Ruby > Demonic Rune (Supermacro)
Requires [FindAndUse](#use-healthstone--root-tuber-supermacro).
```
/run FindAndUse("Mana Ruby", "Demonic Rune")
```

#### PoM + Trinkets + Arcane Power (Supermacro)
For Arcane Power/Frost build, with one or two power trinkets. Will also activate Berserking (Troll racial) and shoot Frostbolt. Spam the button until you see a flying Frostbolt. 
```
/cast Presence of Mind
/run UseInventoryItem(13)
/run UseInventoryItem(14)
/run if not buffed("Arcane Power") then CastSpellByName("Arcane Power") end
/cast Berserking
/cast Frostbolt
```

#### Auto Fireball
Use this as your main Fireball button, to automatically activate trinkets, Berserking (Troll racial) and Combustion, if available. Will **not** show error messages if any of these on cooldown! 👏🏼

Requires clicking multiple times for each spell that is available.
```
/run if GetInventoryItemCooldown("player",13)==0 then UseInventoryItem(13) end
/run if GetInventoryItemCooldown("player",14)==0 then UseInventoryItem(14) end 
/run if GetSpellCooldown(5,'spell')==0 then CastSpellByName("Berserking") end 
/run if GetSpellCooldown(88,'spell')==0 then CastSpellByName("Combustion") end
/cast Fireball
```

#### Auto Frostbolt
Use this as your main Frostbolt button, to automatically activate trinkets if available. Will **not** show error messages if any of them on cooldown! 👏🏼

Requires clicking multiple times for each trinket that is available.
```
/run if GetInventoryItemCooldown("player",13)==0 then UseInventoryItem(13) end
/run if GetInventoryItemCooldown("player",14)==0 then UseInventoryItem(14) end 
/cast Frostbolt
```

## Warrior

#### Bloodthirst / Heroic Strike
Cursesy of Cosa. Change 47 to 50 if you dont have talents in HS, requires [GetSpellId](#get-spell-id-by-name-supermacro) or you replace it with the concrete spell ID.
```
/cast Bloodthirst
/run local autoattack=GetSpellId("Auto Attack") if not IsCurrentAction(autoattack) then UseAction(autoattack) end
/run if UnitMana("player")>=47 then CastSpellByName("Heroic Strike") else CastSpellByName("Bloodthirst") end
```
