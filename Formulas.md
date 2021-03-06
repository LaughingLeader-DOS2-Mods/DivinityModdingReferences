# Divinity: Original Sin 2 - Definitive Edition Formulas <!-- omit in toc -->

Big thanks to Norbyte for finding all of these.

# Table of Contents
- [Table of Contents](#table-of-contents)
  - [Accuracy](#accuracy)
    - [AccuracyBonusAbility](#accuracybonusability)
    - [WeaponAccuracyPenalty](#weaponaccuracypenalty)
    - [WeaponAccuracy](#weaponaccuracy)
    - [Accuracy](#accuracy-1)
    - [DodgeBoost](#dodgeboost)
    - [Dodge](#dodge)
    - [HitChance](#hitchance)
  - [Armor Scaling](#armor-scaling)
  - [Character Stats](#character-stats)
    - [Attribute Growth](#attribute-growth)
  - [Ability Growth](#ability-growth)
  - [Skill Heal Scaling](#skill-heal-scaling)
  - [Vitality Scaling](#vitality-scaling)
  - [Pickpocket Pricing](#pickpocket-pricing)
  - [Trap Wits](#trap-wits)

## Accuracy

### AccuracyBonusAbility

```c++
if (!Weapon) AccuracyBonusAbility = None;
elseif (LeftWeapon && RightWeapon && LeftWeapon.IsRanged == RightWeapon.IsRanged) AccuracyBonusAbility = DualWielding;
elseif (Weapon == BOW || Weapon == CROSSBOW || Weapon == CUSTOM) AccuracyBonusAbility = Ranged;
elseif (Weapon.IsTwoHanded) AccuracyBonusAbility = TwoHanded;
else AccuracyBonusAbility = SingleHanded;
```
[Back to top](#Table-of-Contents)

### WeaponAccuracyPenalty

| Data Key | Default Value |
| ------------- | ------------- |
| WeaponAccuracyPenaltyPerLevel | -20 |
| WeaponAccuracyPenaltyCap | -80 |

```c++
if (Weapon.Level > Character.Level)
    WeaponAccuracyPenalty = min(WeaponAccuracyPenaltyPerLevel * (Weapon.Level - Character.Level), WeaponAccuracyPenaltyCap);
else
    WeaponAccuracyPenalty = 0;
    
if (!Weapon->TwoHanded)
    WeaponAccuracyPenalty = WeaponAccuracyPenalty / 2;
```
[Back to top](#Table-of-Contents)

### WeaponAccuracy

| Data Key | Default Value |
| ------------- | ------------- |
| CombatAbilityAccuracyBonus | 5 |

```c++
if (AccuracyBonusAbility == SingleHanded)
    AccuracyBonus = round(Character.AbilityLevel[AccuracyBonusAbility] * CombatAbilityAccuracyBonus);
else
    AccuracyBonus = 0;

WeaponAccuracy = Character.Accuracy + (sum(Accuracy) of all active potion effects) + (sum(Accuracy) of all equipped items) + WeaponAccuracyPenalty + AccuracyBonus;
```
[Back to top](#Table-of-Contents)

### Accuracy

| Data Key | Default Value |
| ------------- | ------------- |
| TalentPerfectionistAccuracyBonus | 10 |

```c++
BaseAccuracy = max(LeftHandWeaponAccuracy, RightHandWeaponAccuracy);
if (Character.Talent[Perfectionist])
    BaseAccuracy = BaseAccuracy + 1 + TalentPerfectionistAccuracyBonus;

ChanceToHitBoost = (sum(ChanceToHitBoost) of all active potion effects) + (sum(ChanceToHitBoost) of all equipped items);
Accuracy = min(Accuracy + ChanceToHitBoost, 100);
```
[Back to top](#Table-of-Contents)

### DodgeBoost

| Data Key | Default Value |
| ------------- | ------------- |
| AttributeBaseValue | 10 |
| DodgingBoostFromAttribute | 0 |

```c++
DodgeBoost = round((Character.Finesse - AttributeBaseValue) * (DodgingBoostFromAttribute * 100));
```
[Back to top](#Table-of-Contents)

### Dodge

| Data Key | Default Value |
| ------------- | ------------- |
| CombatAbilityDodgingBonus | 1 |

```c++
Dodge = Character.Dodge + (sum(Dodge) of all active potion effects) + (sum(Dodge) of all equipped items) + Character.DodgeBoost;
if (AccuracyBonusAbility == DualWielding)
    Dodge += round(CombatAbilityDodgingBonus * Character.AbilityLevel[DualWielding]);
if (!Character.Talent[Flanking] && Character.Flanks > 1)
    Dodge -= round((Character.Flanks - 1) * FlankedPenalty);
if (Character.HasTalent[Dwarf_Study])
    Dodge += 5;
if (Character.HasTalent[DualWieldingDodging])
    Dodge += 10;
```
[Back to top](#Table-of-Contents)

### HitChance

```c++
targetDodge = 0;
if ( !Target.IsIncapacitated )
  targetDodge = Target.Dodge;
baseHitChance = min(round((100.0 - targetDodge) * Attacker.Accuracy / 100.0), 100.0);
HitChance = min(baseHitChance + Attacker.ChanceToHitBoost, 100.0)
```
[Back to top](#Table-of-Contents)

## Armor Scaling

| Data Key | Default Value |
| ------------- | ------------- |
| AttributeBaseValue | 10 |
| ExpectedConGrowthForArmorCalculation | 1 |
| VitalityBoostFromAttribute | 0.07 |
| ArmorToVitalityRatio | 0.55 |

```c++
armorScaling = (vitalityBoost * ((AttributeBaseValue + level * ExpectedConGrowthForArmorCalculation - AttributeBaseValue) * VitalityBoostFromAttribute) + 1.0) * ArmorToVitalityRatio;
armor = armorScaling * armor / 100;
```
[Back to top](#Table-of-Contents)

## Character Stats

### Attribute Growth

| Data Key  | Default Value |
| ------------- | ------------- |
| AttributeBoostGrowth  | 0.75  |

```c++
ceil(((AttributeValue - 11) / 10 * Level) * AttributeBoostGrowth)
```
[Back to top](#Table-of-Contents)

### Attribute Growth (Level Mapped)

| Data Key  | Default Value |
| ------------- | ------------- |
| AttributeBoostGrowth  | 0.75  |
| AttributeLevelGrowth  | 2 |
| AttributeGrowthDamp   | 0.7 |
| AttributeBaseValue    | 10 |

```c++
(ceil((AttributeValue - 1) / 100.0 * (AttributeLevelGrowth + AttributeBoostGrowth) * Level) * AttributeGrowthDamp) + AttributeBaseValue
```
[Back to top](#Table-of-Contents)

## Ability Growth

| Data Key  | Default Value |
| ------------- | ------------- |
| CombatAbilityNpcGrowth  | 0.1  |
| CombatAbilityCap  | 10  |

```c++
min(round(Level * AbilityValue * CombatAbilityNpcGrowth), CombatAbilityCap)
```
[Back to top](#Table-of-Contents)

## Skill Heal Scaling

| Data Key | Default Value |
| ------------- | ------------- |
| VitalityStartingAmount | 21 |
| VitalityExponentialGrowth | 1.25 |
| VitalityLinearGrowth | 9.091 |
| VitalityToDamageRatio | 5 |
| VitalityToDamageRatioGrowth | 0.2 |
| ExpectedDamageBoostFromAttributePerLevel | 0.065 |
| ExpectedDamageBoostFromSkillAbilityPerLevel | 0.015 |
| ExpectedDamageBoostFromWeaponAbilityPerLevel | 0.025 |
| ExpectedConGrowthForArmorCalculation | 1 |
| FirstVitalityLeapLevel | 9 |
| FirstVitalityLeapGrowth | 1.25 |
| HealToDamageRatio | 1.3 |
| SecondVitalityLeapLevel | 13 |
| SecondVitalityLeapGrowth | 1.25 |
| ThirdVitalityLeapLevel | 16 |
| ThirdVitalityLeapGrowth | 1.25 |
| FourthVitalityLeapLevel | 18 |
| FourthVitalityLeapGrowth | 1.35 |

```c++
vitalityExp = pow(VitalityExponentialGrowth, level - 1);
if ( level >= FirstVitalityLeapLevel )
{
vitalityExp = vitalityExp * FirstVitalityLeapGrowth / VitalityExponentialGrowth;
}
if ( level >= SecondVitalityLeapLevel )
{
vitalityExp = vitalityExp * SecondVitalityLeapGrowth / VitalityExponentialGrowth;
}
if ( level >= ThirdVitalityLeapLevel )
{
vitalityExp = vitalityExp * ThirdVitalityLeapGrowth / VitalityExponentialGrowth;
}
if ( level >= FourthVitalityLeapLevel )
{
vitalityExp = vitalityExp * FourthVitalityLeapGrowth / VitalityExponentialGrowth;
}
vitalityBoost = roundf((level * VitalityLinearGrowth) + (VitalityStartingAmount * vitalityExp)) / 5 * 5.0;

levelScaledDamage = vitalityBoost / (((level - 1) * VitalityToDamageRatioGrowth) + VitalityToDamageRatio);

averageLevelDamage = (((level * ExpectedDamageBoostFromSkillAbilityPerLevel) + 1.0) * levelScaledDamage)
       * ((level * ExpectedDamageBoostFromSkillAbilityPerLevel) + 1.0);

return round(healValue * averageLevelDamage * HealToDamageRatio / 100.0);
```
[Back to top](#Table-of-Contents)

## Vitality Scaling

| Data Key | Default Value |
| ------------- | ------------- |
| VitalityStartingAmount | 21 |
| VitalityLinearGrowth | 9.091 |

```c++
vitalityBoost = roundf((level * VitalityLinearGrowth) + (VitalityStartingAmount * vitalityExp)) / 5 * 5.0;
result = vitalityBoost * vitality / 100
```
[Back to top](#Table-of-Contents)

## Pickpocket Pricing

| Data Key | Default Value |
| ------------- | ------------- |
| PickpocketExperienceLevelsPerPoint | 4 |
| PickpocketGoldValuePerPoint | 200 |
| FirstPriceLeapLevel | 9 |
| FirstPriceLeapGrowth | 1.75 |
| SecondPriceLeapLevel | 13 |
| SecondPriceLeapGrowth | 1.15 |
| ThirdPriceLeapLevel | 16 |
| ThirdPriceLeapGrowth | 1.5 |
| FourthPriceLeapLevel | 18 |
| FourthPriceLeapGrowth | 1.15 |
| GlobalGoldValueMultiplier | 1 |

```c++
PickpocketExpLevel = round(PickpocketSkill * PickpocketExperienceLevelsPerPoint);
priceGrowthExp = pow(PriceGrowth, PickpocketExpLevel - 1);
if ( PickpocketExpLevel >= FirstPriceLeapLevel )
{
  priceGrowthExp = priceGrowthExp * FirstPriceLeapGrowth / PriceGrowth;
}
if ( PickpocketExpLevel >= SecondPriceLeapLevel )
{
  priceGrowthExp = priceGrowthExp * SecondPriceLeapGrowth / PriceGrowth;
}
if ( PickpocketExpLevel >= ThirdPriceLeapLevel )
{
  priceGrowthExp = priceGrowthExp * ThirdPriceLeapGrowth / PriceGrowth;
}
if ( PickpocketExpLevel >= FourthPriceLeapLevel )
{
  priceGrowthExp = priceGrowthExp * FourthPriceLeapGrowth / PriceGrowth;
}
price = ceil(PickpocketGoldValuePerPoint * priceGrowthExp * GlobalGoldValueMultiplier);
return 50 * round(price / 50.0);
```

# Trap Wits

| Data Key | Default Value |
| ------------- | ------------- |
| AttributeBaseValue | 10 |
| AttributeLevelGrowth | 2 |
| AttributeBoostGrowth | 0.75 |
| WitsGrowthDamp | 0.5 |

```c++
if ( difficulty > 1 )
{
	result = (ceil((((difficulty - 1) / 100.0) * (AttributeLevelGrowth + AttributeBoostGrowth)) * level) * WitsGrowthDamp) + AttributeBaseValue;
}
else
{
	result = AttributeBaseValue;
}
```

[Back to top](#Table-of-Contents)
