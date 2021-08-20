This mod overrides the normal two Titanium you get upon scanning a fragment when you already have its blueprint and gives components of that blueprint instead... with some exceptions.
Some advanced materials like plasteel and benzene can almost never be salvaged.
Some materials like lubricant are less likely to be salvaged successfully, but not impossible.
Less complex materials like copper wire, lead, or nickel are easy to salvage.

This is my first mod for Subnautica, please be gentle.

Thanks go to MrPurple6411 for pointing me to his Copper from Scanning code that I used as a foundation to build this on.

Known Bugs: If, between picking up a chip from a data box for already-owned tech and the end of the animation, the crosshair is moved to a fragment, components will be given as if that fragment was scanned. MrPurple6411 informs me that there is no way to fix this bug without using a transpiler, which is beyond my abilities at present.

As of 1.0.1, the mod now has configuration support! No, please, hold your applause.
In-game, there is now an option for "Minimum ingredients" and "Maximum ingredients"; Upon scanning a known fragment, a number of components between these two values (inclusive) will be given, up to a limit of the number of ingredients in the recipe; this means that even if minimum and maximum are both set to 10, if the recipe only has three ingredients you'll only get three ingredients.

Aside: any values over 1 minimum and 3 maximum seem excessive to me, but if you, the end user, want to go balls to the wall, who am I to stop you?

However, what couldn't really work in the GUI (as far as I know) are two additional parameters: TechWeights, and SubstitutionList. These will be found in a file called config.json in the directory with the DLL file. If you're already familiar with JSON then you're halfway there, but if you're not I'd suggest finding your own tutorial on the subject; such a tutorial would be beyond the scope of this document.

SubstitutionList is applied first, so we'll cover it first. Once the mod has built a list of recipe ingredients, it checks them against the SubstitutionList to see if it should be swapped. SubstitutionList takes a form like so:
"SubstitutionList": [
	{
		"replacedTech": "AdvancedWiringKit",
		"replacements": [
			{
				"techType": "Silver",
				"amount": 2
			},
			{
				"techType": "ComputerChip",
				"amount": 1
			}
		]
	},
	{
		"replacedTech": "PlasteelIngot",
		"replacements": [
			{
				"techType": "ScrapMetal",
				"amount": 2
			},
			{
				"techType": "Lithium",
				"amount": 2
			}
		]
	},
	{
		"replacedTech": "TitaniumIngot",
		"replacements": [
			{
				"techType": "ScrapMetal",
				"amount": 2
			}
		]
	}
]

replacedTech is the TechType (more on that later) for the item that should be replaced. replacements is a table that the code reads when an item in a recipe matches replacedTech. replacements can have multiple replacements, as shown above. So if item requires an Advanced Wiring Kit, this will be replaced with two pieces of silver ore (TechType of just 'Silver') and one ComputerChip.

For example; if you scan a nuclear reactor fragment, the recipe starts out as PlasteelIngot, AdvancedWiringKit, Lead, Lead, Lead. After substitution with the above list, this recipe becomes ScrapMetal, ScrapMetal, Lithium, Lithium, Silver, Silver, ComputerChip, Lead, Lead, Lead.

Once substitution has been processed, TechWeights comes into play. TechWeights looks a little like this:

"TechWeights": {
	"MercuryOre": "0",
	"SandLoot": "0",
	"TitaniumIngot": "0",
	"AdvancedWiringKit": "0",
	"PlasteelIngot": "0",
	"EnameledGlass": "0.5",
	"HydrochloricAcid": "0.5",
	"AminoAcids": "0",
	"Polyaniline": "0.5",
	"Nanowires": "0",
	"Lubricant": "0.5"
}
Each line takes the form "TechType": "weight" where weight is a value that affects the probability that this particular item will be picked. Any TechType not present in the table receives a value of 1, so TechWeights only needs to include items that should not drop, should drop less often, or should drop more often.

This time we'll use a Seamoth fragment as our example. After substitution, the recipe becomes ScrapMetal, ScrapMetal, PowerCell, Glass, Glass, Lubricant, Lead. The components not in TechWeights receive a value of 1.0, so in the end we have:
ScrapMetal 1.0
ScrapMetal 1.0
PowerCell 1.0
Glass 1.0
Glass 1.0
Lubricant 0.5
Lead 1.0.

This means that Lubricant has half the chance of dropping on any given roll as any other component. It's still possible, but not as likely.

Note that after each drop roll - the exact number of such drops being picked randomly between the "minimum ingredients" and "maximum ingredients" chosen in the GUI - whichever item is rolled is removed from the list before the next roll. If our first roll is Lead, then at the start of the second roll, the list is now
ScrapMetal 1.0
ScrapMetal 1.0
PowerCell 1.0
Glass 1.0
Glass 1.0
Lubricant 0.5
so Lubricant is now slightly-more likely to drop than it was on the first roll. But still unlikely.

TechTypes
TechTypes are a class of data used by the game to assign values to all kinds of spawnable tech; this includes inventory items, flora, fauna, blueprints and more. Unless you're a coder you probably don't need any more than that. However, if you want to edit the tech weights or substitution list, you will need to know what the names of the TechTypes involved are. The Subnautica Fandom wiki can be used to find the name of specific items - look for the entry Debug Spawn for this name - but currently-unknown is whether case sensitivity is a thing. (For example, table coral samples are listed as "jeweleddiskpiece" on the Wiki but JeweledDiskPiece in the class; I don't currently know whether 'jeweleddiskpiece' would work, or whether you must use JeweledDiskPiece as listed in the class.)

The TechTypes class, listing every TechType in the vanilla game, is included in the download from 1.0.1. The numbers aren't actually important for this mod but you should keep the case (upper/lower/mixed) of the name in mind. Modded TechTypes will require you to do your own research. SMLHelper includes an option to help with this; in the Options menu, under SMLHelper, set "Extra item info" to "Mod name and item ID"; then, when your mouse hovers over an item in-game, the item's name will be listed in the tooltip. This works on Habitat Builder pages too.

Aside: This documentation took me a LOT longer to write than the code did.

v1.1: No more included configs
As of Version 1.1.0.0, a config.json has not been included in the download; this is to prevent any settings configured by the user being overwritten every time. However, this does mean any new values will not be initialised. It may be best, between versions, to back up your existing config.json, allow the game to generate a new one, and update that with the versions from your previous config.

v1.1: RecipeOverrides
For when overriding individual components isn't enough, RecipeOverrides, new in v1.1, allows you to override the entire recipe for a given fragment. The default value for this property is as follows:
"RecipeOverrides": [
	{
		"overridden": "CyclopsHullFragment",
		"replacements": [
			{
				"techType": "PlasteelIngot",
				"amount": 3
			},
			{
				"techType": "Lead",
				"amount": 2
			}
		]
	},
	{
		"overridden": "CyclopsBridgeFragment",
		"replacements": [
			{
				"techType": "EnameledGlass",
				"amount": 3
			},
			{
				"techType": "Lead",
				"amount": 1
			},
			{
				"techType": "PlasteelIngot",
				"amount": 1
			}
		]
	},
	{
		"overridden": "CyclopsEngineFragment",
		"replacements": [
			{
				"techType": "Lubricant",
				"amount": 1
			},
			{
				"techType": "AdvancedWiringKit",
				"amount": 1
			},
			{
				"techType": "Lead",
				"amount": 2
			},
			{
				"techType": "PlasteelIngot",
				"amount": 1
			}
		]
	},
	{
		"overridden": "CyclopsDockingBayFragment",
		"replacements": [
			{
				"techType": "PlasteelIngot",
				"amount": 3
			},
			{
				"techType": "Lead",
				"amount": 1
			}
		]
	}
],
If upgrading from an earlier version, it may be simplest to copy and paste the above block into your existing config.json; Paste it just after the opening curly bracket, at the top.
For each entry in this list, overridden is the name of the fragment to be overridden. If the player scans a fragment whose TechType matches this parameter, then its default recipe, if it has one, is ignored and replaced with the contents of replacements. replacements is built just as it is in SubstitutionList, just that this list replaces all of the entries in the original recipe, not one.

RecipeOverrides is processed before TechWeights and SubstitutionList, meaning that all items on the Recipe Override are still subject to being substituted or having their weight zeroed out.

v1.1: Mod support
As of v1.1, modded TechTypes should be supported, including those added by CustomCraft2