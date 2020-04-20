---
title:  Coding tips and debugging your script files: Tips on how to not fuck up your code 
author: Bashtime
date: 20.04.2020
category: Scripting
---

In this guide I will cover some basic mistakes one might do while trying to code custom items in lua. Also I want give some tips how to give your code a better structure.


##Missing "" can hang up your game forever
For example the command AddNewModifier wants the name of the modifier and not the modifier itself. So if you wonder why your game starts to hang as soon as you buy a certain item or learn a custom ability that uses this command, pay attention if you didn't miss the quotation marks.  

##Make sure special values of one ability in your npc_abilities_custom file or npc_items_custom don't have the same number.
This mistake might happen when you copy parts of one spell that you need in another spell as well. For example all the stuff from Mekansm is included in guardian greaves.
			"01"
			{
				"var_type"				"FIELD_INTEGER"
				"bonus_ms"				"60"
			}
			"02"
			{
				"var_type"				"FIELD_INTEGER"
				"bonus_mana"			"250"
			}
			"01"
			{
				"var_type"				"FIELD_INTEGER"
				"aoe_armor"			"3"
			}
This mistake can make your game hang forever even though in the kv-checker the file looks completly fine.

##Careful when you copy paste stuff
Most times many things stay the same when you want to create a new ability or item. The block below is needed for every custom-made lua item. 

		"ID"							"1363"														// unique ID number for this item.  Do not change this once established or it will invalidate collected stats.
		"AbilityBehavior"				"DOTA_ABILITY_BEHAVIOR_IMMEDIATE | DOTA_ABILITY_BEHAVIOR_NO_TARGET"
		"AbilityTextureName"			"item_arcane_boots"
		"BaseClass"						"item_lua"
		"ScriptFile"					"items/greaves_classic"
    
You dont want to write the same stuff over and over again. That's why we copy abilities and only change the necessary stuff to speed up our coding.
Sometimes you could be too fast and copy whole items twice and not notice. This will cause an infinite hang on your machine and you'll wonder what went wrong in your life. Now you might have found an answer.


##Infinite Loops by making copying mistakes again
Everytime an ability or item has an innate modifier or applies a modifier you will need LinkLuaModifier(modifierName, saveLocation, LUA_MODIFIER_MOTION_NONE)
to tell the machine where it can find the modifier for your ability. To not have like 1k files, you can put the modifier code inside the ability code. An Example:

item_buckler_classic = class({})

LinkLuaModifier("modifier_buckler_classic","items/buckler_classic", LUA_MODIFIER_MOTION_NONE)

function item_buckler_classic:GetIntrinsicModifierName()
	return "modifier_buckler_classic"
end

Make sure that scriptFile A doesnt refer to scriptFile B and vice versa to use a certain modifier. This will cause an infinite loop and crash your game.
So pay attention you changed the saveLocation properly after copying.


##Local definitions can be used outside of functions in the scriptFile
Letz have a look at 2 items that I created. Both items use the same functions. All I need to change is the class if I want to do a 3rd item.

-- Buckler Passive Bonuses Modifier
modifier_buckler_classic = class({})

--------------------------------------------------------------------------------
-- Classifications
function modifier_buckler_classic:IsHidden()
	return true
end

function modifier_buckler_classic:IsPurgable()
	return false
end



-------------------------------------------------------------------------------
-- Kaya and Sange Passive Bonuses Modifier

modifier_kns_classic = class({})

--------------------------------------------------------------------------------
-- Classifications
function modifier_kns_classic:IsHidden()
	return true
end

function modifier_kns_classic:IsPurgable()
	return false
end



Instead of replacing the modifier in 10+ functions we define a local variable that makes the code easier to re-use for other items as well.
By making it a little more abstract we save hours of unnecessary work. 

Example: Optional stuff can be deleted; This can be used as a template

--Class Definitions

	item_aquila_classic = class({})
	local itemClass = item_aquila_classic

	--Aura Bonuses Modifier
	modifier_aquila_aura = class({})
	local buffModifierClass = modifier_aquila_aura
	local buffModifierName = 'modifier_aquila_aura'
	LinkLuaModifier(buffModifierName, "items/aquila_classic", LUA_MODIFIER_MOTION_NONE)	

	--Passive instrinsic Bonus Modifier
	modifier_aquila = class({})
	local modifierClass = modifier_aquila
	local modifierName = 'modifier_aquila'
	LinkLuaModifier(modifierName, "items/aquila_classic", LUA_MODIFIER_MOTION_NONE)


		--Usual Settings
		function itemClass:GetIntrinsicModifierName()
			return modifierName
		end

		function modifierClass:IsHidden()
			return true
		end

		function modifierClass:IsPurgable()
			return false
		end

		function modifierClass:RemoveOnDeath()
    		return false
		end

		function modifierClass:GetAttributes()
			return MODIFIER_ATTRIBUTE_MULTIPLE
		end


			--Optional Aura Settings
			function modifierClass:IsAura()
    			return true
			end

			function modifierClass:IsAuraActiveOnDeath()
    			return false
			end
				--Who is affected ?
				function modifierClass:GetAuraSearchTeam()
    				return DOTA_UNIT_TARGET_TEAM_FRIENDLY
				end

				function modifierClass:GetAuraSearchType()

						--For Toggle
						if self:GetAbility():GetToggleState() then  
							return DOTA_UNIT_TARGET_HERO
						end 

						return (DOTA_UNIT_TARGET_HERO + DOTA_UNIT_TARGET_BASIC)
				end

				function modifierClass:GetAuraSearchFlags()
    				return DOTA_UNIT_TARGET_FLAG_NONE
				end

			function modifierClass:GetAuraRadius()
				local radius = self:GetAbility():GetSpecialValueFor( "radius" )
    			return radius
			end

			function modifierClass:GetModifierAura()
    			return buffModifierName
			end


    --Active Modifier (here: Tempory Armor Bonus when using Mekansm)
		modifier_mekansm_active = class({})
		local modifierAClass = modifier_mekansm_active
		LinkLuaModifier("modifier_mekansm_active", "items/mekansm_classic", LUA_MODIFIER_MOTION_NONE)

		function modifierAClass:IsHidden()
			return false
		end

		function modifierAClass:IsPurgable()
			return true
		end

		function modifierAClass:RemoveOnDeath()
    		return true
		end


		function modifierAClass:DeclareFunctions()
			local funcs = {
				MODIFIER_PROPERTY_PHYSICAL_ARMOR_BONUS,
			}
			return funcs
		end

			function modifierAClass:GetModifierPhysicalArmorBonus()
				return self:GetAbility():GetSpecialValueFor("heal_armor")
			end



--Active item part

function itemClass:OnSpellStart()

	-- unit identifier
	local caster = self:GetCaster()
  
end



##Use the same specialValue names for the same things in every ability or item to save time.
Getting a nil-value from a non-defined special variable causes no problem for the game. However if you defined it your ability will automatically grant you these bonuses without causing any work



-----------------------------------------
--Passive Modifier Stuff starts here

function modifierClass:OnCreated()

	-- common references (Worst Case: Some Nil-values)
	self.bonus_dmg = self:GetAbility():GetSpecialValueFor( "bonus_dmg" )
	self.bonus_armor = self:GetAbility():GetSpecialValueFor( "bonus_armor" )
	self.bonus_ms = self:GetAbility():GetSpecialValueFor( "bonus_ms" )
	self.bonus_as = self:GetAbility():GetSpecialValueFor( "bonus_as" )
	self.bonus_mr = self:GetAbility():GetSpecialValueFor( "bonus_mr" )

	self.bonus_str = self:GetAbility():GetSpecialValueFor( "bonus_all_stats" )
	self.bonus_agi = self:GetAbility():GetSpecialValueFor( "bonus_all_stats" )
	self.bonus_int = self:GetAbility():GetSpecialValueFor( "bonus_all_stats" )

	self.bonus_hp = self:GetAbility():GetSpecialValueFor( "bonus_hp" )
	self.bonus_mana = self:GetAbility():GetSpecialValueFor( "bonus_mana" )

	self.hp_reg = self:GetAbility():GetSpecialValueFor( "hp_reg" )
	self.mana_reg = self:GetAbility():GetSpecialValueFor( "mana_reg" )	

	-- Extra references

end

			--------------------------------------------------------------------------------
			-- Modifier Effects
			function modifierClass:DeclareFunctions()

				local funcs = {

					--The Usual Modifiers
					MODIFIER_PROPERTY_PREATTACK_BONUS_DAMAGE,
					MODIFIER_PROPERTY_PHYSICAL_ARMOR_BONUS,
					MODIFIER_PROPERTY_MOVESPEED_BONUS_CONSTANT,
					MODIFIER_PROPERTY_MOVESPEED_BONUS_PERCENTAGE,
					MODIFIER_PROPERTY_ATTACKSPEED_BONUS_CONSTANT,
					MODIFIER_PROPERTY_MAGICAL_RESISTANCE_BONUS,

					MODIFIER_PROPERTY_STATS_STRENGTH_BONUS,
					MODIFIER_PROPERTY_STATS_AGILITY_BONUS,
					MODIFIER_PROPERTY_STATS_INTELLECT_BONUS,

					MODIFIER_PROPERTY_HEALTH_BONUS,
					MODIFIER_PROPERTY_MANA_BONUS,
					MODIFIER_PROPERTY_HEALTH_REGEN_CONSTANT,
					MODIFIER_PROPERTY_MANA_REGEN_CONSTANT,

					--Add more stuff below

				}

				return funcs
			end

				--DMG; ARMOR; MS; AS; MR
				function modifierClass:GetModifierPreAttack_BonusDamage()
					return self.bonus_dmg
				end

				function modifierClass:GetModifierPhysicalArmorBonus()
					return self.bonus_armor
				end

				function modifierClass:GetModifierMoveSpeedBonus_Constant()
					return self.bonus_ms
				end

				function modifierClass:GetModifierAttackSpeedBonus_Constant()
					return self.bonus_as
				end

				function modifierClass:GetModifierMagicalResistanceBonus()
					return self.bonus_mr
				end



				--STR; AGI; INT
				function modifierClass:GetModifierBonusStats_Strength()
					return self.bonus_str
				end

				function modifierClass:GetModifierBonusStats_Agility()
					return self.bonus_agi
				end

				function modifierClass:GetModifierBonusStats_Intellect()
					return self.bonus_int
				end



				--HP; MANA; REG
				function modifierClass:GetModifierHealthBonus()
					return self.bonus_hp
				end

				function modifierClass:GetModifierManaBonus()
					return self.bonus_mana
				end

				function modifierClass:GetModifierConstantHealthRegen()
					return self.hp_reg
				end

				function modifierClass:GetModifierConstantManaRegen()
					return self.mana_reg
				end



--#########################
--## Aura Stuff starts here
--#########################

function buffModifierClass:OnCreated()

	--Common References
	self.aoe_reg = self:GetAbility():GetSpecialValueFor( "aoe_reg" )
	self.aoe_armor = self:GetAbility():GetSpecialValueFor( "aoe_armor" )
end

function buffModifierClass:DeclareFunctions()
	local funcs = {
		MODIFIER_PROPERTY_PHYSICAL_ARMOR_BONUS,
		MODIFIER_PROPERTY_HEALTH_REGEN_CONSTANT,
	}
	return funcs
end

function buffModifierClass:GetModifierPhysicalArmorBonus()
	return self.aoe_armor
end

function buffModifierClass:GetModifierConstantHealthRegen()
	return self.aoe_reg
end


##Useful unlisted commands
Trying to reproduce the old iron Talon I ran into a huge problem. I didnt know how to check if my target was a tree or a unit.
This is the solution: IsInstance()
See the application below.

function itemClass:OnSpellStart()
	
	-- unit identifier
	local caster = self:GetCaster()
	local target = self:GetCursorTarget()

	self.isTree = target:IsInstance(CDOTA_MapTree) --Checks if the target is a tree, but not every tree is CDOTA_MapTree
  
	if (not self.isTree) then

		local stillTree = (target:GetClassname() == "dota_temp_tree") --Checks again if it's still a tree, LOL  
    
    
  Note: You need to Kill() temp_tree (iron branch tree, sprout tree and ironwood tree), whereas MapTrees have to get CutDown() to be removed.
  
  
  
  
  
  





Hope this will help you in your future codings ;)


















