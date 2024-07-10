# Lowman Central API
The official Lowman API for Destiny 2. It will provide you with any Lowman data you would wish for to use in your projects. API is free to use, however The only requirement is that you include the attribution "Data provided by lowman-central.com" in a visible and accessible location within your project.

# Endpoints

- The main URL for accessing endpoints is `api.lowman-central.com`. Certain endpoints may utilize a different URL to optimize server load, which will be specified where applicable.



# `/getPlayerData`

The `/getPlayerData` endpoint returns every Lowman Raid that a player has completed based on the provided `membershipID` in the header. Take note that Lowman Central only stores completed activities.

## Endpoint Usage
### Request

GET /getPlayerData


Headers:
 - membershipID: <player's membership ID> `(string)`

Replace `<player's membership ID>` with the actual membership ID of the player. This header must be included in the request. If an invalid `membershipID` or a player that doesn't exist in our database is provided, an empty array will be returned.

## Response Information
An array of objects will be returned with the following keys:

- **instanceID**: PGCR identifier for the activity. This value can be used to make a request to Bungie API to get more information.
- **raid**: Possible values include: edge, crota, ron, kf, vow, vog, dsc, gos, lw, pantheon, crown, scourge, eater, or levi.
- **difficulty**: Indicates the difficulty of the activity (legend or master); Exception is Pantheon which will either have atraks, oryx, rhulk or nezarec.
- **fireteamSize**: Indicates the size of the fireteam (1, 2, or 3; 4 is possible only for Salvation's Edge).
- **playdate**: Date of the activity without the time component.
- **playtime**: Time spent in the instance in seconds.
- **isFlawless**: Boolean indicating if the activity was completed flawlessly (no player deaths).
- **isFresh**: Boolean indicating if the activity is fresh.

## Response Example
```json
[
    {
        "instanceID": 14080604157,
        "raid": "crota",
        "difficulty": "legend",
        "fireteamSize": 3,
        "playdate": "2023-11-24T00:00:00.000Z",
        "playtime": 7374,
        "isFlawless": 0,
        "isFresh": 1
    },
    {
        "instanceID": 14036663447,
        "raid": "crota",
        "difficulty": "legend",
        "fireteamSize": 3,
        "playdate": "2023-11-07T00:00:00.000Z",
        "playtime": 4704,
        "isFlawless": 0,
        "isFresh": 1
    }
]
```



# `/getUserCharacters`

The `/getUserCharacters` endpoint retrieves all characters associated with a player's `membershipID` provided in the header. This endpoint optimizes profile loading in Lowman Central by storing the each player's character latest PGCR raid in the database. This approach eliminates the need to repeatedly parse their complete activity history from the Bungie API, significantly improving performance for profiles which have a lot of raid clears. You could use this endpoint as a faster way to get all characters a user has, or if you want to parse raid activityHistory for a player faster. 

## Endpoint Usage
### Request

GET /getUserCharacters


Headers:
  - membershipID: <player's membership ID>

Replace `<player's membership ID>` with the actual membership ID of the player. This header must be included in the request. If an invalid `membershipID` or a player that doesn't exist in our database is provided, an JSON with "error" key and it's value "Profile not found" will be returned. This endpoint returns an array of objects with the following keys:

## Response Information
This endpoint returns an array of objects with the following keys:

- **characterID**: Unique identifier for a character.
- **latestPGCR**: PGCR identifier of a latest raid this character was in.
- **status**: Boolean indicating if the character is currently active or deleted.


## Response Example
```json
[
    {
        "characterID": "2305843009310867556",
        "latestPGCR": "15161510307",
        "status": 1
    },
    {
        "characterID": "2305843009299550445",
        "latestPGCR": "15153919252",
        "status": 1
    },
    {
        "characterID": "2305843009333884718",
        "latestPGCR": "15142303810",
        "status": 0
    }
]
```

# `/getPlacementFullClears`

The `/getPlacementFullClears` endpoint retrieves up to one hundred placements for fresh full clears, based on specified criteria.  Lowman Central placements are based on `unique Fireteams`, to prevent `placement farming`.
## What is considered a unique Fireteam?
A Fireteam is considered unique if it contains same players.


Let's say we have a Fireteam of these three players:
- Ducko
- Shloob
- Mojo  

These three players have thirty clears of a requested Raid Placement. For this Fireteam only the earliest clear will be counted for their placement, and all the other clears will not be anywhere in the placements.
Now let's say Mojo had to leave and another player hopped in, so the Fireteam contains these players:
- Ducko
- Shloob
- Bandit  

Even though Ducko and Shloob already have a placement with Mojo, the third player is different, therefore this is considered as a unique Fireteam and their earliest clear will appear as their placement.
In cases where a certain player (let's say Ducko for example) has multiple placements on the leaderboards, their best placement will be shown in their profile (for example if Ducko has #3 and #14, #3 will be their placement).
The same logic is applied to Duo Fireteams.

## Endpoint Usage
### Request

POST /getPlacementFullClears


Body:
  - raid: edge, crota, ron, kf, vow, vog, dsc, gos, lw, pantheon, crown, scourge, eater, or levi `(string)`
  - difficulty: legend, master and atraks, oryx, rhulk or nezarec in case of Pantheon `(string)`
  - fireteamSize: 1,2,3 or 4 (four only possible for Salvations's Edge) `(int)`
  - isFlawless: 1 or 0 `(int)`
  - page: make it 1 for the first 100 placements `(int)`



## Response Information
The `structure` of the returned JSON `varies depending` on whether you're requesting `Solo placements or Duo and Trio placements`.

### Response for Solo Placements
- **instanceID**: PGCR identifier for the activity. This value can be used to make a request to Bungie API to get more information.
- **playerName**: Player's BungieID (Example#0000). If the user doesn't have a unique BungieID which was given in Season of the Lost, it will return "JohnDoe#9999".
- **membershipIDs**: Unique identifier for the player.
- **membershipType**: Platform identifier of the player. In case where Cross Save is enabled, this is their main platform.
- **playdate**: Date of the activity without the time component.
- **emblemHash**: Hash of Player's Equiped Emblem at the end of the activity. Image accessed via `https://www.bungie.net/common/destiny2_content/icons/<emblemHash>.jpg`, where instead of `<emblemHash>` you would enter this value.
- **isProMember**: Boolean indicating if the player has an active Lowman Central Pro subscription.

### Response for Trio/Duo Placements
- **instanceID**: PGCR identifier for the activity. This value can be used to make a request to Bungie API to get more information.
- **bestTime**: Time spent in the instance in seconds. (Ignore the name itself, it's not getting their best time, I was just reusing some code and being lazy lol).
- **playerNames**: Each Fireteam Player's BungieIDs (Example#0000). If a player doesn't have a unique BungieID which was given in Season of the Lost, their name will be set to "JohnDoe#9999".
- **membershipIDs**: Unique identifier for each player in the Fireteam, ordered by playerNames.
- **membershipTypes**: Platform identifiers of the players, separated by commas. Ordered by playerNames. In case where Cross Save is enabled, this is their main platform.
- **emblemHashes**: Hash of each Player's Equiped Emblem at the end of the activity, ordered by playerNames. Images accessed via `https://www.bungie.net/common/destiny2_content/icons/<emblemHash>.jpg`, where instead of `<emblemHash>` you would enter this value.
- **isProMembers**: Boolean values indicating if the player has an active Lowman Central Pro subscription, ordered by playerNames.
- **earliestDate**: Date this activity took place without the time component (once again weird name, but i reused some code, sorry lol).


## Response Examples

### Solo Placements
```json
[
    {
        "instanceID": 14023936966,
        "playerName": "Dabs#5809",
        "membershipIDs": "4611686018434710196",
        "membershipType": 1,
        "playdate": "2023-11-02T00:00:00.000Z",
        "emblemHash": "fc8f0fda624220dbd8c85e92e7c5cbf6",
        "isProMember": 0
    },
    {
        "instanceID": 14334376950,
        "playerName": "Bog on my dog#7426",
        "membershipIDs": "4611686018490167330",
        "membershipType": 3,
        "playdate": "2024-01-15T00:00:00.000Z",
        "emblemHash": "3c251b702026fee24488eac5cbd3a2e2",
        "isProMember": 0
    }
]
```
### Trio/Duo Placements
```json
[
    {
        "instanceID": 12715533070,
        "bestTime": 2885,
        "playerNames": "aeiouuuuu#9829,Azteec#9197,Marlin#8844",
        "membershipIDs": "4611686018486233550,4611686018453633505,4611686018510489736",
        "membershipTypes": "2,1,3",
        "emblemHashes": "35ad1236cc60b36c160dde8e3fda90aa,35ad1236cc60b36c160dde8e3fda90aa,35ad1236cc60b36c160dde8e3fda90aa",
        "isProMembers": "0,0,0",
        "earliestDate": "2023-03-13T00:00:00.000Z"
    },
    {
        "instanceID": 12717181663,
        "bestTime": 2475,
        "playerNames": "Adrenaline#5985,basic#7840,Flaux#554",
        "membershipIDs": "4611686018470036385,4611686018445837471,4611686018446563602",
        "membershipTypes": "2,2,2",
        "emblemHashes": "c65ef1f06bcee6e3a38f789dea000f13,69ce6a6c823503026d1c6466a76ecb0e,70038c6d33c095f109a9df3f2bfc28de",
        "isProMembers": "0,0,0",
        "earliestDate": "2023-03-13T00:00:00.000Z"
    },
]

```

# `/getPlacementBosses`

The `/getPlacementBosses` endpoint retrieves up to one hundred placements for Boss Checkpoints, based on specified criteria. These placements are used on Lowman Central only if the entire raid isn't clearable for the difficulty. For example, Root of Nightmares doesn't have Solo Nezarec placement because it can be fully Solo Flawlessed. Lowman Central placements are based on `unique Fireteams`, to prevent `placement farming`.
## What is considered a unique Fireteam?
A Fireteam is considered unique if it contains same players.


Let's say we have a Fireteam of these three players:
- Ducko
- Shloob
- Mojo  

These three players have thirty clears of a requested Raid Placement. For this Fireteam only the earliest clear will be counted for their placement, and all the other clears will not be anywhere in the placements.
Now let's say Mojo had to leave and another player hopped in, so the Fireteam contains these players:
- Ducko
- Shloob
- Bandit  

Even though Ducko and Shloob already have a placement with Mojo, the third player is different, therefore this is considered as a unique Fireteam and their earliest clear will appear as their placement.
In cases where a certain player (let's say Ducko for example) has multiple placements on the leaderboards, their best placement will be shown in their profile (for example if Ducko has #3 and #14, #3 will be their placement).
The same logic is applied to Duo Fireteams.

## Endpoint Usage
### Request

POST /getPlacementFullClears


Body:
  - raid: edge, crota, ron, kf, vow, vog, dsc, gos, lw, pantheon, crown, scourge, eater, or levi `(string)`
  - difficulty: legend, master and atraks, oryx, rhulk or nezarec in case of Pantheon `(string)`
  - fireteamSize: 1,2,3 or 4 (four only possible for Salvations's Edge) `(int)`
  - page: make it 1 for the first 100 placements `(int)`


Incorrectly entering the request body or if there is no data for the request, JSON will be returned with a key "message" and value "No leaderboard data found for the specified criteria".

## Response Information

- **instanceID**: PGCR identifier for the activity. This value can be used to make a request to Bungie API to get more information.
- **playerNames**: Each Fireteam Player's BungieIDs (Example#0000), separated by commas. If a player doesn't have a unique BungieID which was given in Season of the Lost, their name will be set to "JohnDoe#9999".
- **membershipIDs**: Unique identifier for each player in the Fireteam, ordered by playerNames and separated by commas.
- **membershipTypes**: Platform identifiers of the players, ordered by playerNames and separated by commas. In case where Cross Save is enabled, this is their main platform.
- **emblemHashes**: Hash of each Player's Equiped Emblem at the end of the activity, ordered by playerNames and separated by commas. Images accessed via `https://www.bungie.net/common/destiny2_content/icons/<emblemHash>.jpg`, where instead of `<emblemHash>` you would enter this value.
- **isProMembers**: Boolean values indicating if the player has an active Lowman Central Pro subscription, ordered by playerNames and separated by commas.
- **playdate**: Date this activity took place without the time component.


## Response Example
Unlike the `/getPlacementFullClears` endpoint, the response will always have the same format regardless of the Fireteam size.

```json
[
    {
        "instanceID": 12715533070,
        "bestTime": 2885,
        "playerNames": "aeiouuuuu#9829,Azteec#9197,Marlin#8844",
        "membershipIDs": "4611686018486233550,4611686018453633505,4611686018510489736",
        "membershipTypes": "2,1,3",
        "emblemHashes": "35ad1236cc60b36c160dde8e3fda90aa,35ad1236cc60b36c160dde8e3fda90aa,35ad1236cc60b36c160dde8e3fda90aa",
        "isProMembers": "0,0,0",
        "earliestDate": "2023-03-13T00:00:00.000Z"
    },
    {
        "instanceID": 12717181663,
        "bestTime": 2475,
        "playerNames": "Adrenaline#5985,basic#7840,Flaux#554",
        "membershipIDs": "4611686018470036385,4611686018445837471,4611686018446563602",
        "membershipTypes": "2,2,2",
        "emblemHashes": "c65ef1f06bcee6e3a38f789dea000f13,69ce6a6c823503026d1c6466a76ecb0e,70038c6d33c095f109a9df3f2bfc28de",
        "isProMembers": "0,0,0",
        "earliestDate": "2023-03-13T00:00:00.000Z"
    },
]

```

# `/getLatestLowmanClear`

The `/getLatestLowmanClear` is used to get the latest Lowman that has been completed. 

## Endpoint Usage
### Request

GET /getLatestLowmanClear


No Headers are required.


## Response Information
This endpoint returns an array of an object with the following keys:

- **instanceID**: PGCR identifier for the activity. This value can be used to make a request to Bungie API to get more information.
- **raid**: Possible values include: edge, crota, ron, kf, vow, vog, dsc, gos, lw, pantheon, crown, scourge, eater, or levi.
- **difficulty**: Indicates the difficulty of the activity (legend or master); Exception is Pantheon which will either have atraks, oryx, rhulk or nezarec.
- **isFresh**: Boolean indicating if the activity is fresh.
- **isFlawless**: Boolean indicating if the activity was completed flawlessly (no player deaths).
- **fireteamSize**: Indicates the size of the fireteam (1, 2, or 3; 4 is possible only for Salvation's Edge).
- **playerNames**: Each Fireteam Player's BungieIDs (Example#0000), separated by §. If a player doesn't have a unique BungieID which was given in Season of the Lost, their name will be set to "JohnDoe#9999".
- **membershipIDs**: Unique identifier for each player in the Fireteam, ordered by playerNames and separated by §.
- **membershipTypes**: Platform identifiers of the players, separated by §. Ordered by playerNames. In case where Cross Save is enabled, this is their main platform.
- **emblemHashes**: Hash of each Player's Equiped Emblem at the end of the activity, ordered by playerNames and separated by §. Images accessed via `https://www.bungie.net/common/destiny2_content/icons/<emblemHash>.jpg`, where instead of `<emblemHash>` you would enter this value.
- **isProMembers**: Boolean values indicating if the player has an active Lowman Central Pro subscription, ordered by playerNames.



## Response Example
```json
[
    {
        "instanceID": 15276691882,
        "raid": "vog",
        "difficulty": "legend",
        "isFresh": 1,
        "isFlawless": 0,
        "fireteamSize": 3,
        "playerNames": "swift#7100§Greazy_Gibby#6123§HungryItalian#6538",
        "membershipIDs": "4611686018494026861§4611686018522462044§4611686018476228718",
        "membershipTypes": "1§2§3",
        "emblemHashes": "a78c6bdc5a3772ca76c37f6ab18a9d40§90437a7df0b4337e06ed7e7a94a9828c§c362efc1d99ecd70b9df07b3b00623c3",
        "isProMembers": "0§0§0"
    }
]

```

