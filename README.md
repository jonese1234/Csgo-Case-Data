# Csgo-Case-Data
This is that data package for https://cases.jonesy.moe/

## Data format changes
As of `2023-02-28 23:50:39Z` the data format has changed to include multiple skin market places other than steam.

## How the statistics for the CSGO case details are calculated
### Background
This is the explanation about how [my website](https://cases.jonesy.moe/) calculates the statistics shown. 
### The Numbers & Data
Before i can show you how i calculate the statistics i need to show where the data is coming from.  
#### Skin prices
Firstly the skin data is all collected from the steam market place every 8hs. I do not collect this data myself but use [csgotrader.app's](https://csgotrader.app/) API which can be found here [csgotrader.app/prices](https://csgotrader.app/prices/).
#### Rarity Probability 
The actual drop rate for the skins raritys have publicly been released by valve when they catered to the Chinese csgo market as Chinese laws state that any gambling/case unboxing has to have the odds publicly available. Information on this can be [found here.](https://www.reddit.com/r/GlobalOffensive/comments/6zd9yx/perfect_world_csgo_has_finally_published_their/)     
The Results of which are:
|Rarity  | Standard  | StatTrak |
|--|--|--|
|Special items  | 0.25575%  |0.02558% |
| Covert | 0.63939%  |0.06394% |
| Classified | 3.19693%  | 0.31969%|
| Restricted | 15.98465% | 1.59847%|
| Mil-Spec | 79.92327% | 7.99233%|

#### Wear Probability 
Unlike rarity the exact probability for skin wear has not been publicly reviled by Valve. This means we have to calculate this ourselves. Luckily Reddit user [Step7750](https://www.reddit.com/user/Step7750) has [done that here.](https://www.reddit.com/r/GlobalOffensiveTrade/comments/dx035d/psa_an_analysis_on_the_weird_distribution_of/) Utilizing the 250 million items in the CSGOFloat database the values below where created.
Results in this table: 
|Factory New (0.00-0.07)|\~3%|
|:-|:-|
|Minimal Wear (0.07-0.15)|\~24%|
|Field-Tested (0.15-0.38)|\~33%|
|Well-Worn (0.38-0.45)|\~24%|
|Battle-Scarred (0.45-1.00)|\~16%|
Now we have all this data we can create the statistics.

### Calculating the statistics
For demonstration sake we will be using CS:GO Weapon Case 3. The Data used can be [found here.](https://gist.github.com/jonese1234/80764bede6191003fe7c63a044469e57)

#### Step 1. Calculate the average price for each skin.
For this example we will take the CZ75-Auto | Tread Plate. 
Data:

    {
		"CZ75-Auto | Tread Plate": {
			"wears": ["FN", "ST FN", "MW", "ST MW", "FT", "ST FT"],
			"prices": {
				"FN": 2.93,
				"ST FN": 11.94,
				"MW": 2.18,
				"ST MW": 6.62,
				"FT": 2.07,
				"ST FT": 4.34
			},
			"rarity": "Restricted"
		}
	}, 
I use this example to show the 1 fall down of the way i calculate results. 
Lets start with Factory New. This skin come with a standard and a StatTrak variant. StatTrak variants drop at a 10% chance. 

Now before we do anything due to this skin not having Well Worn or Battle scared we need to make some adjustments to the the wear table above. This is the only downside with this current approach due to not having the exact float values. 
What we do is we take these missing values and apply them to the other skin wear probability.

    Well Worn + Battle Scared = 0.4
    0.4 / 3 = 0.133333
    Factory new now = 0.03 + 0.13333 = 0.16333333
While this may not be super accurate it is one way to deal with it.

We then times these new values by the percentage chance of getting them. 0.9 (90%) for the normal and 0.1 (10%) for the StatTrak.

    Normal: 0.9 * 0.16333333 = 0.147
    StatTrak: 0.1 * 0.16333333 = 0.016333333333333335
    
We need to take the price of the Normal skin ($2.93) and times by the wear percentage value (0.16333333 or 16.333%) located in the above and repeat for the StatTrak variant. 

    Normal: 2.93‬ * 0.147 = 0.42923999999999995
    StatTrak: 11.94 * 0.016333333 = 0.19453000000000004
   
 We now repeat this step for each wear value. Returning the results below.
 

    FN: 0.42923999999999995
    FN ST: 0.19453000000000004
    MW: 0.73248
    MW ST: 0.24639999999999995
    FT: 0.8673600000000001
    FT ST: 0.20201333333333338
You then sum this result to get the average expected price of the CZ75-Auto | Tread Plate.
Returning a value of **$2.6720233333333336**

Repeat all these steps for each skin. You then get [the results here.](https://gist.github.com/jonese1234/6e2e79e8e72518abcb2ac303d6bfa51e)

#### Step 2. Calculate the average prices of each rarity.
This is a simple step compared to the previous one.

Just mean average each rarity type. For Mill-Spec it would look like this.

     --- Data ---
    'CZ75-Auto | Crimson Web': 1.4048299999999998,
    'P2000 | Red FragCam': 0.40520999999999996,
    'Dual Berettas | Panther': 0.49759000000000014,
    'USP-S | Stainless': 5.282,
    'Glock-18 | Blue Fissure': 1.7008500000000002,
    
    --- Calculation ---
    1.4048299999999998 + 0.40520999999999996 + 
    0.49759000000000014 + 5.282 + 1.7008500000000002 = 9.29048
    
    9.29048 / 5 = 1.858096
    
So for Mill-Spec the **average skin is $1.86.**
Now repeat for each rarity.

#### Step 3. Calculate average overall skin price.
After you have all the average skin rarity values you have to take a look at the first table. This contains the probability of you getting a skin of that type.
  
    'mill-spec': 1.858096,
    'restricted': 2.8584508333333334,
    'classified': 13.292348333333333,
    'covert': 9.429039999999999,
    'special': 200.03564163333334

So you need to times each by there respective probability.

    'mill-spec': 1.858096 * 0.7992327 = 1.4850510829392
    'restricted': 2.8584508333333334 * 0.1598465 = 0.4569133611304167
    'classified': 13.292348333333333 * 0.0319693 = 0.4249470715728333
    'covert': 9.429039999999999 * 0.0063939 = 0.06028833885599999
    'special': 200.03564163333334 * 0.0025575 = 0.51159115347725

Now you sum all these values.
**The final result is 2.9387910079757003  or $2.94**

#### Step 4. Calculate Return on Investment (ROI)
This is just as simple as the last.
Take the final result and divide by the cost of the case + key.

    2.9387910079757003 / (1.37 + 2.49) = 0.76134 or 76.13%

 **So the ROI is 76.13%** however this does not take into account Steam Market fees. 
 
