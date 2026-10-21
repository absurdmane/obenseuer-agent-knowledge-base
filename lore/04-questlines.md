# Obenseuer: Questlines

Scope: every quest chain the digests and the task and conversation sources show, each told as a short story: who gives it, the goal, the steps, the branches and consequences, the rewards and what they unlock. Ordered by how the chains connect to the main arc; side quests are marked. Objective and flag ids are given where the sources name them. Where a chain is known only from conditions, that is said.
Status: unverified source (digested by a model from game text, not played)
Related: the other lore documents
**SPOILERS: this document tells the whole story of the game.**

How to read this: the sources name tasks in two ways. The task texts (localization file) give titles and sometimes short descriptions. The conversations show the steps through flags such as ObjectiveActive, ObjectiveCompleted and TaskActive. Many descriptions are empty, so the steps below come partly from scripts and conditions, and those are marked as such. A "branch" is a dialogue choice that changes a flag or an outcome.

---

## Part A. The main chains

### A1. The abandoned tenement (opening, main)

**Giver**: the intake interviewer [83], or the interviewers in the opening cutscene [415].
**Goal**: understand the tenement you were given and decide what to do with it.
**Steps**: the task "Abandoned tenement" [376, 377] has three sub-tasks: examine the apartments [378], examine the stairwell [380], find the tenement [382]. The player also gets a map.
**Branches**: the intake asks personal questions (the answers are logged; Anya later has the database of them). The cutscene path starts with the red-rock dig and the revoked relocation [415].
**Rewards and unlocks**: unlocks the renovation and tenant chains (A2), the rental flags, and the first passport problem (A3).

### A2. Renovation and tenants (Malone chain, main)

**Givers**: Malone Petronis [4]; contractor Samuel Jonasson [82].
**Goal**: renovate the building so tenants can move in, and rent the apartments.
**Steps** (tasks [456]-[476], [572]-[576]):
1. Find a tenant for one apartment [456].
2. "Getting started" [458]: Malone offers to help renovate; Samuel Jonasson in Deekula can do the work.
3. Ask Samuel about the renovation [462]; pay the first installment; buy materials from the O-Market stalls or the One Stop Shop; build the apartment walls first so the first tenant can move in. Renovation takes days and you cannot enter while it is being done.
4. Report back to Malone [464] when the renovation is done.
5. "Start five renovations" [466], "Start renovation" [468], and tell Malone about the renovation or renovations [470, 472].
6. Tell Malone about the rental [474] (his flags: MaloneRent, ResidentRentedAmount).
7. Malone sends the player to a black-bearded man, Pentti, for work [476].
8. Kurahaara farm [460]: Malone says the player needs a lot of money and should ask for a job at the Kurahaara farm (see B-section, farm chain).
9. "My Crazy Neighbor" [572]: a neighbour has moved into one of the player's apartments. Options: get him to pay rent, evict him, or talk to the police about him.
10. "My cozy apartment" [574]: Malone says to take the first-floor apartment; then [COPYRIGHT] [576].

**Branches**: Malone can be evicted (the relay evict script) if the player's answers go badly. He can also leave or be beaten (variables MaloneBeaten, MaloneIsLeaving). He offers janitor help after apartment levels 3 and 4 (see B9).
**Rewards and unlocks**: rent income (the main income path, per the menu settings) and janitor services. Renovation levels unlock the Crazy Neighbor and janitor chains.

### A3. Passports and the Bazaar gate (main)

**Givers**: Malone (who wants the player to have an Obenseuer passport) [440]; Passmore [72]; Eliana Passmore [145], [178]; Hank Möbelmann [58], [108].
**Goal**: a passport that the Bazaar gate accepts.
**Steps** (tasks [430]-[454], [384]-[396], [438]-[440], [612]-[614]):
1. Find a passport seller (Deekula C, [436]) and find someone who sells them [440].
2. Find Passmore, say the password ([COPYRIGHT], [432]; or [COPYRIGHT], [434]). Passmore lives in Tenement B, behind the O-Market.
3. Passmore refuses, says he no longer sells, and sends the player to Eliana [72].
4. Ask Eliana to forge a passport [430]. She wants help first: the priest's debt in Deekula B1 [612]. Eliana sends the player to the priest's flat; the player finds him dead or near death (see the priest's flat, A4).
5. Eliana takes the photo [446]; the passport arrives by mail in a couple of days [454].
6. "Getting a passport" [448]. With the forged passport (ForgedPassport11) the player can pass Axel at the gate.
7. Alternative routes: David Lund sells hand-made passports (100 RM, no refund) [71]; Hank sells a route for about 100 RM [58]; Passmore's route needs two tasks (see A4).

**Branches**: refusing the priest task means Eliana will not forge. A hand-drawn passport is rejected at the mine management office, where Tanja says it looks child-made [207:66].
**Rewards and unlocks**: Bazaar entry; the Redemption door key with proof of residence (Fidan, 500 donations); the bank and mining permit checks.

### A4. The Bank of Bazaar heist and the Mailanen twist (main)

**Givers**: Passmore [72] (BazaarBank33, PassmoreReport26, BankHunt77).
**Goal**: rob the Bank of Bazaar through the Bergmann water tunnels.
**Steps** (tasks [384]-[396], [438]-[452], [694]-[690]):
1. Get the passport from Eliana (A3), then return to Passmore about the bank [384].
2. [COPYRIGHT] [438]: find the bank entrance in the Bergmann tunnels. The tunnel door's keypad is [COPYRIGHT], and the entrance is in the building where he lives. The bank drain cover is only open from 23:00 to 03:00 [388].
3. Verify the bank tunnel reaches the bank: FoundBank17. The drain cover does not budge at first; a buddy is to unblock it at 23:00.
4. [COPYRIGHT] [386]: four sticks (BankDynamite32). Passmore later says only one is needed.
5. "Open the vault" [390]: the boss's room upstairs has the cameras; they can be closed from there. The vault has an alarm if it is blown open.
6. Tell Passmore what you found [392], [452]: someone robbed the bank before you. This is the Heikki reveal (BankFound34).
7. [COPYRIGHT] [394].
8. [COPYRIGHT] [396] (BankPolice43). The police call Passmore a suspicious man and then warn the player to stay out of illegal business [88:47-48].
9. [COPYRIGHT] [694]: Passmore's friend Heikki Mailanen emptied the bank. Talk to a bank employee [690] or the electronics salesperson [692] (Jonas Elström) to find Heikki.

**Branches**: the player can refuse, join, threaten to go to the police, or ask to be paid to keep quiet. Paying Passmore gives a small share. Passmore hands over half the money once the player brings it [72:159]. Passmore names Heikki [72:165].
**Consequences**: Heikki is hunted by the casino owner (see A5). Passmore resents the bank manager and Heikki. The police's reaction is cynical.

### A5. Casino takeover (main, linked to A4)

**Givers**: Casper Colikke [348] (casino owner); Heikki Mailanen [431]; the Crazy Neighbor [434]; Seamus Liepa [444]. Objectives: casinoCoup10, CasperRid66, CasperCrazy23, BankHunt77.
**Goal**: find Heikki and the stolen money; then decide who runs the casino.
**Steps**:
1. Casper hires the player to find Heikki, who robbed the vault with the manager's help. Raimo already took part of the money [426, 428].
2. Heikki (in Mentti B) says he kept about 20 percent and gave it to Casper as a debt. He blames the manager, asks the player to get rid of Casper (lying, poison or a neighbour's help), then take the casino [431].
3. The player can bring Casper money or excuses. Casper asks for proof: a poisoned Socca Junior bottle, red mushroom extract, mystery meat, a jar of dirt, or the player's own claim to have cut fingers [435].
4. Casper offers the player a security job next to Raimo [435].
5. The Crazy Neighbor takes the casino and hires Casper as a substitute manager. He sets CasinoState to "Crazy" [434].
6. Seamus Liepa forces Casper to sign the casino over and leave Obenseuer (Casper becomes Gone) [444, 445]. Seamus wanted 10000 RM for the job [209].
7. Heikki takes the casino back (CasinoState returns to Normal), pays the player a reward and gives a giant coin mascot [446].

**Branches**: Heikki can be pressured with a threat to his fingers. The sources do not settle which of the casino outcomes is final.

### A6. Anya, Hanss and the gate (main)

**Givers**: Anya [145], [77] (AnyaStart, Glowstick7); Eliana [145].
**Goal**: find Hanss in the mines and get through the gate to "the other side".
**Steps** (tasks [480]-[490], [512], [510], [714]-[720], [698]-[702]):
1. Eliana sends the player to the basement of Deekula B, where Anya is. Anya asks the player to find Hanss. Refusal makes her threatening.
2. [COPYRIGHT] [480]: keys should be in Hanss's place. "Glowstick trail" [482]: Hanss went into the mines with glow sticks; follow them.
3. [COPYRIGHT] [486]: the trail ends at a collapse. [COPYRIGHT] [488].
4. Report back [484]. "Claim the reward" [490]: [COPYRIGHT] text in the task; the player should return to Ville for the reward (the Ville chain, A6-side).
5. [COPYRIGHT] [512] and [COPYRIGHT] [510]: Anya sends the player to Karl Hagen for another route to Hanss.
6. Gate: the player finds a pipe and a huge gate in a cave. The keypad is [COPYRIGHT] (matches the interview number). The player opens it with the Osrat device, which speaks about G.A.L.E.N. Anya had promised the device's maker not to talk about her.
7. [COPYRIGHT] [714]; [COPYRIGHT] [716]: Eliana gives the basement password [COPYRIGHT]. [COPYRIGHT] [720]: Hanss lives there.
8. "Tunnel B2" [698]: Mike's body in the Bergmann tunnels had a key labelled B2. [COPYRIGHT] [696]: a mannequin monster (TunnelB212).
9. [COPYRIGHT] [700], [702]: Mike mentions this book, sold at the Bazaar bookshop or library (links to A11).

**Branches**: Anya does not want the red extract and asks the player not to tell Fredrik about the sample (AnyaFredrikSample). Her knowledge of the player's interview answers is a threat.
**Rewards and unlocks**: the gate, the route to the other side (only partly explored), the Karl Hagen meeting, the Bazaar's library book, the B2 tunnel.

### A7. Red mushrooms and Fredrik's experiment (main)

**Givers**: Fredrik [162], [163], [188] (tasks [650]-[656]); the newsreader [161] reports a body with red mushrooms.
**Goal**: deliver red mushrooms to Fredrik's lab; help with his experiment; take the extract on to Anya.
**Steps**:
1. [COPYRIGHT] [478]: a body with red mushrooms grows near the big underground gate.
2. [COPYRIGHT] [650]: Fredrik asks for help with the mushrooms you collected. "Red Danger" [652]: he wants to know what the red mushrooms are.
3. [COPYRIGHT] [656]: Tenement A apartment 10, via the doorphone.
4. The experiment: the player puts the mushrooms into a device on the operating-room table, presses the start button, takes the red extract, finds an empty syringe, and injects the extract into an IV bag attached to the patient. The patient becomes aggressive and her body cannot take it. Reward offered: 1500 RM.
5. Fredrik's log counts the days since the experiment began (day 146 at one point). A phone in the room lets the player ask to be let out or report losing the extract. Losing the extract means losing the reward and making the test impossible.
6. [COPYRIGHT] [654]: Fredrik sends the extract to Anya in the Deekula B basement.
7. Fredrik's second job: [COPYRIGHT] (A8).

**Branches**: losing the extract (Fredrik says he should have handled it himself); giving the extract to Pia (she dies, A8); reporting the experiment to the police (no result shown).
**Rewards and unlocks**: the 1500 RM reward; Fredrik's Tenement A keys; the Anya link.

### A8. Pia, Alice and Joel (the missing friend, main mystery)

**Givers**: Pia [8]; Fredrik [424]-[428], [163]; Tatyana [666]; Axel [664]. Objectives: Alice35, DeeplyInsane35 (the mines version), PiaPolice88, TellPiaAlice46, AliceShack53, Joelive22, PiaMines23.
**Goal**: find Alice, then find out what happened to Pia and whether Joel is involved.
**Steps** (tasks [424]-[428], [664]-[688]):
1. Bring the medicine to Pia [424]. Pia's friend Alice usually brings it, but Alice has not been seen (Fredrik's patient [426]).
2. Pia asks the player to mail a letter [682] and to find Alice [672], since Alice mentioned a party thrown by Joel [688].
3. Ask Axel [664]: Alice was seen with a man going into Tenement A. Ask Tatyana [666]: Alice has a boyfriend, Joel Hietala.
4. [COPYRIGHT] [674]; "Investigate Joel's apartment" [678]: a key to the small storage next to Pia and Alice's building is found there. [COPYRIGHT] [670].
5. [COPYRIGHT] [680].
6. The storage: a woman's body with a ritual setup, holding a photo of Pia (the body is probably Alice). [COPYRIGHT] [686].
7. [COPYRIGHT] [684]: Pia was found dead in the sauna (per the task text). Other sources say she was in her room, or in a cell with the head taken. Fredrik says the player should call the police (PiaPolice88); the player may refuse [163:146-152].
8. "Call the police" [668] and "Let Amanda know..." etc.: police reports go to 112.
9. [COPYRIGHT] [404], [406] (the second version, [688]): Fredrik says a woman who works in the mines lived with Joel; Ines is the woman in question.

**Branches**: the player can say Alice fled Obenseuer (Pia asks whether she simply left). The player can blame Pia, or blame Joel (a jealous Pia). Pia denies jealousy and says Alice never mentioned a boyfriend. Telling Fredrik that Pia is dead is one option; another is to say the player gave Pia the red extract (Pia dies).
**Consequences**: Pia's head is later reported as taken by the Crazy Neighbor (A11). Joel is seen in Tenement A speaking from a body. Ines's account (A9) gives a version in which Ines killed Alice.

### A9. Ines Hietala and the Stalburg Underground (main, the largest story in the mines)

**Givers**: Ines Hietala [242]-[249] (the mine room); Laura Markkanen [221] and Anna [202] (who point to Ines); the police (DeeplyInsane35, InesPolice33 at Kauha's Claim).
**Goal**: find Ines, get her account, and decide what to do with her.
**Steps**:
1. Find Ines. Anna says she used to work in the canteen with blonde hair [202]. Laura says she was odd and was seen in trouble near Kauha's claim, looking for something [221].
2. Ines, in shock, says she followed "the signs", silenced people and was told what to do by the Stalburg Underground. She says she killed Alice and tries to explain why.
3. Ines offers a drink to "see the truth". Drinking it [246] gives visions (the floating head Kasa and the phone in a hole).
4. Ines's branches: [COPYRIGHT] (she waits for them, or the player calls the police); [COPYRIGHT] (she refuses); axe to the head (requires the axe item; she dies); leaving her alone ("TASK DONE"); asking about an apartment (she does not know where she will be tomorrow); asking about the cult ("NEW TASK": take the mask, go to the church in the swamp, reached through the Bazaar).
5. "After the drink" [246]: Ines wants to find [COPYRIGHT] and asks the player to find them, because they stopped answering her. The task ends with DONE.
6. Conv [249]: Ines's later state, when she must find "it" and go to the gates.

**Branches**: the axe ends her life. The police branch brings officers to Kauha's claim. Taking her to the cult leads to the swamp church.
**Consequences**: Joel and Pia's fate in her account is contradictory (see 06-open-questions). The "mask" in the cult task is not clearly an item.

### A10. The weird strand: Kasa, the phone and the red rocks (main, unclear)

**Givers**: Kasa [243]-[244] (the floating head), the phone voice [245], and the Wasteland.
**Goal**: answer the phone and collect the red rocks.
**Steps**: Kasa says the player made it and that it will explode unless the phone is answered. The phone voice asks whether the mission is done. Each "I failed" leads to the same line: the player was hired because of hope, desperation and stupidity. The mission is to collect the rocks. Conv [250]: touching a red object makes it vanish without a visible change.
**Branches**: answering or refusing the phone; touching the rocks.
**Consequences**: the sources do not state a reward or a failure state. The Welho warnings (A11) tell the player not to listen to the voice or touch the red rocks.

### A11. The Welho path and the Sacred Seal (side for most, but a full chain; objectives TrueWelho11, WaterTrees13, joinpontikka)

**Givers**: Welhö [270], [415]-[418]; Erto Lagerfeld (the Suurwelho) [405]; Mikael Arnaldur [400]; Aune Korsborg [341].
**Goal**: prove you are a true Welho and get the seal, the lair access, and a moonshine recommendation.
**Steps**:
1. Entry: Welhö calls the player a commoner and says they are no true Welho. The card game (B-steps in [416]-[418]) can make the player a Welho if they play well (a Welho hat, respect and recipes). Losing still earns a hat, and an invitation to the Sanctum for rare goods. Giving up makes Welhö call the player an impostor.
2. Welhö sends the player to the Suurwelho, who alone can judge the player and grant the Welho's Mark [270:36-38]. Erto is in a flat on the top floor of Murju B.
3. Erto's trials (in the order the scripts suggest): silence the noisy downstairs neighbour (ReportErto88); fetch the storage item from the Murju cage (SecondMission57, ErtoStorage100), which Alfred's evictions emptied; bring a live pig, 24 bottles of Osmo Olut, a rubber duck, the red hat and his jacket (BringErtoJacket77); then a final trial at the door (Trial55). Erto hands over the Sacred Seal of Welhous [405].
4. With the seal, Aune lets the player into the lair (the seal is checked, WelhoSeal67). Welhö says the player beat the Suurwelho's trials and gives the lair passcode [COPYRIGHT], saying to look at "the well" and that the player needs keys (SellsSecret). A recognised TrueWelho gets special trade and a prize item [270:233-235].
5. The book chain (WaterTrees13, WelhoBook35, MikalFind37): Welhö says he ordered Mikael to burn the last two copies of [COPYRIGHT]. Mikael lives in the slums. The player can collect nine loose pages from Mikael's hut. If the player says Mikael did the job, Welhö rewards the player with item 18010 and 1300 money. If the player says Mikael did not do it (or reports nine unburned pages), Welhö strips Mikael of his title and refuses him elixirs.
6. Moonshine recommendation (joinpontikka10/15/24, welhoShrooms31): Welhö's signature is reserved for true Welhos; he asks for "Mycelial Ambrosia" (mushroom moonshine) of a good quality. Quality decides the reply: flawless (1.75 multiplier), highly acceptable (1.25), not good enough (0.5), literal poison (0). Only a good result gives the recommendation.
7. Quitting: if the player leaves the Welho path, Welhö mocks them as a quitter [270:225].

**Branches**: the card game outcome; the trials; the book pages; the moonshine quality; the Mikael verdict.
**Rewards and unlocks**: the seal, the lair, the Welho hat, the Welho's recipes, and the co-op recommendation. Also the apprentice's punishment.

### A12. The Crazy Neighbor thread (main-adjacent, comic, and linked to Pia and the casino)

**Giver**: the Crazy Neighbor [156], [164], [181], [198], [212], [251], [255]-[257], [434]-[439].
**Goal**: keep the neighbour happy, or get rid of him.
**Steps**:
1. Introduction [156]: repeated trespass warnings end with a teleport (Relay_teleport). Crimes in his presence also trigger a teleport [212].
2. Visits [164]: he asks for three visits a week (a 4-day window is checked in [256]). A visit can include tea, questions, a board game, a cardboard fort and a "teeth collection". Visit count is kept; after more than three visits the flag CanUpgradeApartment is set.
3. Upgrades [164:96-101]: he asks for a balcony. A nicer place makes him visit more.
4. Rent [164:3, 164:11-14]: asking for rent gets a surprise "contribution".
5. Moonshine co-op [164:106-117]: he can arrange recommendation letters (objective joinpontikka); a timer (CN_pontikka_coop_recommendation) must run before the letters arrive.
6. Casino [164:123-139]: he takes the casino job if the apartment has an upgrade (level 2 or a balcony). He grabs his axe and goes. Otherwise he asks for a balcony first.
7. Debt and games [181]: he claims the player owes him a debt that cannot be repaid. A StartEvent cycle reminds the player of the fungal infection.
8. Pia's head [198]: he says he did not kill Pia but recovered her head, and made a lantern from it. He offers a multiple-choice list of who killed her and mocks each choice. Exit lines require Dialogue1-3 to be set.

**Branches**: visits, upgrades, rent, co-op letters, the casino job, and the Pia head.
**Consequences**: the neighbour can take the casino (A5), can be evicted (A2, "My Crazy Neighbor" [572]), and may recommend the player to the co-op (B-section, C5).

### A13. Ending and in-between (the end-states)

This is not a quest but a set of end scenes and hints. See 01-overview.md for the full list. The chain of hints is:
- The shadow shed woman (who had surgery while pregnant) says "find it" and the scene ends [91].
- The shadow priest says the player should not be here [79].
- Anya says the body in the field is the player's own, and the police identified it [77:208-216].
- The SMV ending [84] has the subject in research, with amputations and a red fungus. The subject is moved to quarantine.

---

## Part B. Mines and the Bazaar's connected chains

### B1. The repository gate (Heska, side with main echoes)

**Giver**: Heska Pavonen [225].
**Goal**: enter the radioactive repository.
**Steps**:
1. Clear the collapsed tunnel on the way (Heska thanks the player and gives an item if the player says they did it).
2. Obtain a RepositoryPermit (official-looking papers from Os Mining Services; see B2). Carry a dosimeter. When the dosimeter's Sv reading reaches 1, leave.
3. Heska gives the gate code ([COPYRIGHT]) and unlocks the gate. He says a previous visitor left glow sticks on the route and went toward the other side. The route has collapsed.
4. After the player has been inside, Heska becomes "Scared" and keeps his distance (he will not come near anyone who has been inside).
**Branches**: refusing the code; asking about the radiation.
**Consequences**: a dumping tunnel with a broken cask, a blast door at 50 mSv per hour, the other side. The repository is linked to the Glowstick route and to Toivo's permit (B5).

### B2. The mining permit chain (Reiska, Tanja)

**Givers**: Reijo "Reiska" Rautakaivos [206], [604]; Tanja Nypan [207] (tasks [590]-[606], [408]-[410]).
**Goal**: a mining permit to sell ore and enter the mine.
**Steps**:
1. Talk to Reiska [604]: he buys ore only from miners with a permit.
2. Get the permit from the Management office, next to his desk [596].
3. Fill and return the mining permit form [594]. Tanja finds an error, so the form must be filled again [592]. Bring the permit back to Reiska [590].
4. Pay the miner's tax of 200 RM [598]; "Permission to dig" [600].
5. Show Tanja a valid Bazaar passport [602]; she checks your age. Under 30 and over 70 draw comments.
6. Elevator to the surface [408]: needs permission from the Bazaar. The permissions building is next to the elevator; Reijo says mentioning him may help [410].
7. Repository access: Tanja deals with the RepositoryAccessForm (B1).
**Branches**: Tanja's romance (single or not; coffee or diesel) sets a Status variable; each accepted date adds points, and the greeting later uses the player's first name.
**Rewards and unlocks**: ore sales (Reiska), the surface elevator, the repository permit.

### B3. Ore and claims: Antero and Emil (side, economic)

**Givers**: Antero Erikson [227]; Emil Kauha [228].
**Goal**: mine ore on a claim and deliver it for a share of the receipts.
**Steps**: mine the ore, haul it in tipping wagons or minitrains to the service area, load it into the crushers, collect the receipt, and bring it back. The claim owner pays 20 percent of the receipt value. Emil's claim includes the electrical building, which needs keys: one was taken by a woman with odd questions, one lost by Pertti (objectives KauhaClaimKeys11, KeysBuilding12, ElevatorBuildings11).
**Branches**: Emil allows mushrooms but warns against getting carried away [228:46]. Antero's claim is reached through Erikson's route, which is needed for Ken's tunnel work (B4).
**Rewards**: receipts and the claim's money; keys (the break room has the tenement access keys).

### B4. Ken's tunnel and the tenement's collapse (side, important for the building)

**Givers**: Ken Nakahara [223]; Toivo Malmijoki [224].
**Goal**: clear the unstable tunnel under the player's building so the reinforcement can start.
**Steps**:
1. Toivo says the player owns a mine tunnel in danger of collapse. The player denies owning the tenement; Toivo says the building will collapse without repairs [224:37-38].
2. Ken says the tunnel must be cleared first. He asks the player to clear the path and offers at least 10000 RM [223:25]. The player can refuse, say they are not a professional, or accept.
3. Once cleared, Ken's team reinforces the tunnel. The path runs through Erikson's claim [223:50-72].
4. Toivo says the repository permit depends on the tunnel business (objectives Glowstick7 and HeiskaGate41, Permit101). Toivo gives the player a permit and sends the player to the management office, where "the lady" (Tanja) handles the papers.
**Branches**: refusing; accepting; asking for money.
**Consequences**: the building's safety. If not reinforced, the building falls.

### B5. Toivo's repository permit (side)

Part of B4. The permit lets the player use the repository (B1). The task requires the tunnel business to be done first.

### B6. Mikkel's speed cameras and the minitrain bargain (side)

**Giver**: Mikkel Purola [208].
**Goal**: stop the fines for speeding on the minitrain line.
**Steps**: speeding raises an alert; Mikkel reviews the footage and fines the driver. The player can offer bribes: 100 RM, a 16-pack of Osmo Olut, 200 RM, 2000 RM, 5000 RM, or 10000 RM. Mikkel refuses the money at first and threatens double fines. He is missing one minitrain for his collection, the limited-run Osmo Olut minitrain.
**Branches**: bringing him the Osmo Olut minitrain stops the fines (SpeedCameraDisabled); showing it as a bribe triggers double fines (SpeedCameraDoubled).

### B7. Seamus, Malone and the Orange Heaven body task (main-adjacent)

**Givers**: Malone [4] (Orange Heaven, objectives Orange9, Seamus52, SeamusRing66, PotatoMine23); Seamus [209], [222].
**Goal**: find Seamus Liepa's body, which Malone believes is at the mine; instead find Seamus alive.
**Steps**:
1. "Find Seamus Liepa" [578]: the player reaches the old mine tunnels. [COPYRIGHT] [580]; [COPYRIGHT] [582]. [COPYRIGHT] [584]. "Orange Heaven" [586]: Malone wants the body found; mine access is in the basement of Deekula B.
2. Seamus lets the player in once they explain that Malone wants a body found [222]. Seamus is alive and dislikes being asked about one.
3. [COPYRIGHT] [588]: Seamus is unhappy about Malone. Malone is angry; Seamus is growing potatoes, which Malone mocks as a poor choice of crop.
4. The silver ring (engraved J.P) is the proof of identity. Saunaklonkku, the lake creature, has it. Einar says Seamus wore it at the Sauna Club. Kalle says the potato farmer may have taken it from a dead man. Ivar has not seen a body with one.
5. The Seamus–Malone meeting [210]: Malone is coughing and very ill; he says Harold Rimas took over and that some members left. Seamus says Malone wanted green mushrooms when the orange project failed.
6. Player options: ask Seamus to get rid of Malone; argue Malone is not worth it; tell Malone he will die soon [210:34, 67, 78-82]. Seamus leaves, saying Malone is lucky. Malone's last lines suggest his end, but it is not confirmed.
**Rewards**: Seamus's potatoes (the cheapest in Obenseuer); Seamus's casino job at 10000 RM (A5).

### B8. Mines: lantern factory and other jobs (side)

- **Lantern factory** (Louie Green, Tenement A) [183, 184]: an eight-hour shift at 65 OC per hour; the pay goes up with good work. Blueprints are on the table. The intercom asks whether you are coming to work; the player can refuse or change their mind.
- **Elevator install** [189]: Jonne Reis says Urho Siltanen is the one to ask about installing an elevator in the player's building.
- **Emma-Maija Taylor** [196]: sells cats.

### B9. Janitor work (side)

**Givers**: Malone (after renovation levels 3 and 4) and Lizzy Mopper [213].
**Goal**: keep the building running.
**Steps**: fill the water tanks (including showers and taps) for 435 OC; empty septic tanks for 620 OC; empty sewage tanks under sinks; bring firewood (Petro's firewood can fill a store for 1400 OC).
**Gate**: each service is gated by HasJanitorAction flags and by the renovation level.

---

## Part C. Tenement, Bazaar and local side quests

### C1. Max Masher and the mash ("Me mash!", side with a gate to Arvo)

**Givers**: Pentti Penttilä [33], [56] (tasks [520]-[540]); Adam Holmgren [81] (who says where Max lives).
**Goal**: find Max Masher, who has stopped delivering mash to the speakeasy, and restore the supply.
**Steps**:
1. Find Max in Deekula B, apartment B10 [520, 526].
2. At his door (the "evil door") the player can knock, claim to be police, or claim to be sent by Pentti. Max opens only when mash is mentioned. Kicking the door is part of the punish task [530] (Punish44).
3. Max is hurt. He says the door opened too fast, and wants the door punished (Relay_punish).
4. Ingredients: 10 packs of sugar (item 180050 times 10) and 5 packs of yeast (180060 times 5), green mushrooms, water and malt. The mash goes into a mashing tun through a hatch and comes out into a plastic bucket. The ratio must be correct.
5. Max will help only if paid. The player haggles him up.
6. Pentti pays 300 for ten buckets of sugar mash, 50 for an extra step, and other amounts (70, 360, 550 in the file).
7. Max's phone: the player can return it. Max calls Arvo to recommend the player. The result depends on what the player says about Max's injury. If the player says Pentti sent them to check on Max, Arvo accepts the excuse and says the recommendation must come from Max.
8. Pentti's options (tasks [534]-[540]): tell Pentti that Max can no longer mash; that Max is making mash again; that the player will make mash themselves; or try to get Max to make mash again.
**Consequences**: Pentti starts "fecal kilju" if mash stops; Pentti offers his distillery room behind the bar shelf.

### C2. Pate and the turnip moon (side)

**Giver**: Pate Rantanen [2] (tasks [412]-[416], [616]-[622]).
**Goal**: deliver turnip moonshine.
**Steps**: find the moon [414]; give turnip moonshine to Pate [416] (objectives FindMoon4, PateTunipMoon12). Pate rewards a moon item (AddItemDialogue 300050) and says to keep it.
**Branches**: the player may offer Pate an apartment (26) if the condition allows. Refusing gives a different reply.

### C3. Mr. Duck's ducks and axe (side)

**Giver**: Mr. Duck [7] (tasks [420], [422], [418]).
**Goal**: bring five rubber ducks and an axe.
**Steps**: five live ducks (item 11 times 5; objectives FiveDucks15, Ducks37); an axe (item 10700). He says he butchered most of the ducks with an axe. He threatens to [COPYRIGHT] if refused.
**Branches**: giving the axe ends his plea; refusing makes him quiet. Reward: keys to the upstairs. Advice: not to put ducks in the bathtub.

### C4. Mr. Duck's blue duck (side)

**Giver**: Mr. Duck [93] (tasks [616]-[622]; BlueDuck22, BlueDuckHead22).
**Goal**: find his lost blue rubber duck near the One Stop Shop.
**Steps**: the blue duck is found whole or broken (a missing poster offers a reward but gives no return address). The player can give it back. The reward is a new blue duck and money (200 RM; the player can ask for double). If the duck's head is broken, Mr. Duck reacts angrily and then gives a new duck anyway. The player can keep both ducks or return one.
**Consequences**: "Quack quack!" closes the task.

### C5. The Moonshining Co-op membership (side, with gates)

**Givers**: Arvo Kirkastaja [363]; Aune Kirkastaja [362]; Jeppe Rinkmann [360] (rude doorman); Jooseppi Tuoppi [342] (drinking contest).
**Goal**: become a co-op member, which lets the player sell alcohol in the Bazaar (Astrid Vinsjö) and get equipment.
**Steps** (joinpontikka10, PontikkaEducation10):
1. Pay 100 RM to Arvo for the course (five classes, Tuesday and Thursday at 14:00).
2. Attend the classes and pass the exam (produce moonshine from ingredients on the table). A diploma follows.
3. Apply with three recommendations from current members. Sources name Pentti Penttilä, Max Masher, Malone (for a pack of Norts) and the Crazy Neighbor's letters. Jooseppi writes a recommendation if the player beats him in a drinking contest.
4. Pay the 1300 RM fee for full membership.
5. Membership grants access to the mashing, brewing and distilling equipment, and a hydroponic greenhouse. Price list: Hydroponic Farm 13000 RM (one month), Hydroponic Farm With Supports 10000 RM (one month), Vertical Farm 2500 RM (two weeks), Vertical Farm Box 60 RM (one week). Equipment comes from a company called Närpestech.
**Branches**: a failed course closes the door (TaskFailed joinpontikka10). Max's written letters are unreadable; Arvo accepts Max's word after the call, though he is not fully convinced.

### C6. Redemption and the pea soup (side)

**Givers**: Fidan Tanner [96]; Henrik Korpinen [73]; Tatyana for the directions [46].
**Goal**: pea soup and a Bazaar door key.
**Steps**:
1. Pray before the soup (Henrik): one bowl per day.
2. Donate to Fidan (50 OC or 50 RM each, the counter goes up by 1 or 10). Volunteer for six-hour shifts (adds 50 donations).
3. At 20 points Fidan gives the secret pea soup recipe, which a priest told her about.
4. At 500 donations, with a passport (ForgedPassport11) and Bazaar house keys (bazaarHouseKeys11), the player gets the Bazaar door key (redemptionKeys12) for staff and donors.
5. Henrik gives the home recipe once the first bowl has been given (GaveRecipe).
**Branches**: Kaisa says Fidan eats the soup meant for the needy. Fidan says the government has stopped issuing passports right now.

### C7. The bridge purchase (side, economic)

**Giver**: David Lund [71] (BuyingBridge18, Bridge47, Meet48).
**Goal**: buy the toll bridge in Deekula.
**Offers**: 10 OC or 1000 OC (no deal; asks for RM); 2000 RM for a lifetime pass (no refund); 3000 RM rejected as a joke; 800 RM or 1000 RM gets free entry on the gate. He says he cannot sell the whole bridge, because it has been his only source of income. He offers a lifetime pass for 2000 RM instead.
**Phone**: the toll customer service line refuses complaints (he tells callers to find another route). He asks for the player's name, then tells them to come to flat C6 [111:23].

### C8. Aziz's errands (side, delivery chain)

**Giver**: Akim Azizi [374] (aziz0, aziz10 to aziz100).
**Goal**: deliver food, buy from the meat shop and sabotage a rival.
**Steps**:
1. Buy a döner from Irma Kött (keep the receipt for reimbursement, AzizTaskReceipt).
2. Deliver a combo meal to Virtanen at Murju B flat 9.
3. Deliver a pizza to the casino security guard (Raimo).
4. Deliver a pizza to the jailed inmate Lätty. Riina blocks the pizza at the lobby until the player's bribery check passes, or the warden allows it.
5. Sabotage Scatola di Maccheroni's pizzas by swapping them for bad pizza from the slum hut with the hatch.
**Reward**: a service rifle and a meal. A lost delivery costs 5 RM.

### C9. The jail and the Warden (side, with a hard edge)

**Giver**: the Warden [346], [481], [482]; the prison guard [119]-[126] at the Gatehouse.
**Goal**: avoid prison, or leave it.
**Mechanics**: entering restricted areas sends the player to prison (Relay_Tresspass; TeleportToPrison). Refusing a bag check does the same. Sentences are set by the judge (GetSentenceLength); the player can pay a fine (PayFine) or bribe (CalculateBribe, with multipliers 0.7, 1, 1.25).
**Warden sales**: when a need stat is high enough the warden sells or gives items: a can opener, a duck (DuckNeed over 50), medicine or bandages (Bleeding over 5 or Health under 35), mushrooms (MushroomNeed over 85). Release sends items to storage across the lobby.
**Guard TV**: turning off the guard's TV teleports the player to prison, and the guard is angry.
**Colton's hole** [127]: Colton says a lockpick is under the bed, and the vent opens with it.

### C10. Anna's rats and Kisu (side)

**Giver**: Anna Markkanen [202] (RatProblem74, CaughtRats16; tasks [642]-[646]).
**Goal**: catch six rats in the canteen.
**Steps**: Anna offers two rat traps (items 16120 times 2 and 16130 once). The player may catch rats by hand, or refuse. Reward: a free sandwich and cash.
**Kisu branch**: killing the cat by accident (Anna says she cannot forgive it) or deliberately (Anna says it is the last thing she will say to the player). Kisu accepts fish, a live chick or a rat as a gift.

### C11. Jomppe's farm (side)

**Giver**: Jomppe Kurahaara [87] (JomppeStart, JomppeNoFinnish, Farmer1; tasks [704]-[714], [460]).
**Goal**: grow vegetables in a greenhouse and sell them to Jomppe for cash.
**Steps**: grow cucumbers [704], peppers [706], potatoes [708] and turnips [710]. Special orders: 20 cucumbers (Cucumbers1), 10 peppers (Peppers2), 30 potatoes (Potatoes3). Jomppe accepts only plants grown in the greenhouse. The stove must stay warm, or the plants die of cold. Plants that need support: peppers, cucumbers, peas, tomatoes. Potatoes grow from potatoes.
**Branches**: the Finnish branch when the player says they do not understand Jomppe.

### C12. The Bazaar's bank card (side, paperwork)

**Givers**: the Stalburg Bank customer service [110] and the forms (tasks [542]-[566]).
**Goal**: get a new debit card after the old one is lost.
**Steps**: call the bank; the representative is unhelpful. Write a letter asking for the right form (print or handwrite it) and post it in the mailbox near the O-Market. Show a passport or other proof of living in Obenseuer. The bank first sends a form for closing the whole account; ask again. Fill and send the form. Wait for the debit card. A card may be lost at the border inspection (per "Missing debitcard" [566]). Check the balance at an ATM.
**Reward**: GiveRecipe 10249 (a recipe handed out at the end of the call).
**Branches**: phone card requests are refused for security reasons.

### C13. Mail and the letter from Pia (side)

Part of A8. The letter is mailed at the O-Market mailbox.

### C14. The Chamber of Commerce's expansion (side, building)

**Givers**: Jussi Pöhinä [453], Vanja Bjorkman [456], Karl Hagen [386], Katriina Vermund [454].
**Goal**: upgrades that the Chamber needs before its shop opens.
**Steps**: facade level 3, courtyard level 2, apartment walls level 1. The Chamber hands out a card for a fungal eradication unit and a flyer for a structural contractor [456]. The shop is not released in the files.
**Vanja's wish list**: clear the slums, move people to the swamp, build a bridge checkpoint and a wall, and run canal tours. Jussi warns the government would dislike expansion.

### C15. Samuel Godman's legal remedies (side)

**Giver**: Samuel Godman [299] (ClearCrime_All; UnfreezeAccount_Stalburg Bank).
**Goal**: clear a crime with the police or militia, or unfreeze a bank account (about a week).
**Steps**: invoices go to the player's address; an unpaid invoice blocks further service.

### C16. Open Homes, the luxury house and apartment sales (side)

**Giver**: Anni Kiinteistö [461].
**Goal**: buy a luxury house in the Bazaar (220 000 RM) and understand the listing rules for selling apartments.
**Steps**: selling an apartment needs a notarised deed (from the Land and Property Register, central district), blueprints and an energy certificate. Almost every building lacks the energy certificate.
**Phone**: a property phone line also offers the house (objective sellTheApartments; newBusiness11).

### C17. Fungispring (side)

**Giver**: the Fungispring phone line [463].
**Goal**: order a mushroom-growing cabinet (12500 RM each, one month delivery). Business customers only.
**Note**: the player gives the address [COPYRIGHT] for the order.

### C18. Telemarketing (side, job)

**Giver**: OS magazine office in Kolhola [128] (JobTelemarketer).
**Goal**: earn money by selling subscriptions.
**Steps**: 8-hour shift starting between 6 and 12, paid 45 OC per hour; the subscription costs 64 OC per week, delivered by mail. A good caller can earn hundreds of OC per hour. Subscribers are offered bonuses that do not exist.
**Note**: Fenella and Moray work here.

### C19. Swamp Publishing and Hobbyprof subscriptions (side)

**Giver**: Swamp Publishing phone [193]; Hobbyprof [235].
**Goal**: subscribe to or cancel the Open Sewer magazine, Puuhari (24 RM per month) or ARG (12 RM per month).
**Requirement**: an address with the TenementBlueprint item.

### C20. Samuel Jonasson's tenement blueprint (side, part of A2)

Covered under A2.

### C21. Dr. Pena, the pharmacy and prescriptions (side)

**Givers**: Dr. Pena [92] (turn ticket item 13410); Mona Hall [89] (MonaHallStart).
**Goal**: get a prescription (sleeping pills, painkillers, antidepressants, fungicide, mushroom substitute).
**Steps**: take a turn ticket, see the doctor, then bring the prescription to Mona. Fungicide is weak against the fungal disease.
**Branches**: harassing Mona brings the guards.

### C22. Ivan, Larry, Sirpa and other Bazaar jobs (side)

- **Larry Goatbridge** [297]: machinist paying 22 RM per hour for skilled workers.
- **Sirpa Kvenlayson** [298]: sewing shop, eight-hour day at 10 RM plus 100 OC per hour, with bonuses.
- **Casino security** (Raimo) [351]: eight-hour shift at 140 OC per hour.
- **Dolores Bolognese** [355]: cook at 21 RM per hour or dishwasher at 110 OC per hour.
- **Muscle Max** [284]: fights on Friday to Sunday after 18:00.

### C23. Sauna contest and the samurai (side)

**Givers**: Timo Löyly [387]-[392]; Matti Kiehinen [393]-[398]; Raimo Stalreg [376], [399] (TheLostAngrySamurai, FindRaimo12).
**Goal**: the sauna challenge, and the samurai's locker.
**Steps**: Raimo is blocked from his locker by Matti. The player can bribe Matti for 10 RM, ask him to move, or go into the men's room. Raimo is found and thanked [399]. The sauna challenge with Matti or Timo can end with Timo dead. The spa then closes and staff write a report [380, 390]. Timo's body offers an axe option framed as a joke.
**Also**: Erkki Kianen's group asks the player to find Raimo, who stormed off [328]-[330].

### C24. The Murju pigs and the storage cage (side, part of the Welho path)

Covered under A11.

### C25. Dock and the hippie (side)

**Giver**: a hippie on a phone line [104] (Hippie, HippieTalked; tasks [568]-[570]).
**Goal**: find his lost pink dock (a boat, "she"), taken by people who smelled of sewer and booze.
**Reward**: a special tea recipe and money. Unclear whether the dock is found.

### C26. Mr. Bag and the penny shop (side)

**Giver**: Mr. Bag [85] (MrBagWhatAreYou, MrBagDead, MrBagPunched).
**Goal**: trade with an odd robotic shopkeeper. He dies after being punched three times.

### C27. The Osmo Olut Minitrain purchase (side)

**Giver**: Adam Holmgren [81] (MikkelTrain11, OsmoOlutMinitrain22).
**Goal**: buy the Osmo Olut miniature for 2500 OC after the Event2 flag is set. The purchase triggers Relay_minitrain. It is also the item Mikkel wants (B6).

### C28. The hot dog, the pizza and Sam Morton (side)

- **Sam Morton** [134]: wants late-night pizza; Aziz's deliveries have stopped. Pays real money.
- **Ahmed Rahman** [310]: sells hot dogs.

### C29. Krista Annina and the black world (side)

**Giver**: Krista Annina [36] (Black world whisperers; tasks [36], [162]; variables KristaAnninaReplies, TalkedDream).
**Goal**: enter the black world with her to free the trapped souls under the ground.
**Steps**: she asks for ten pouches of the greenest mushroom (item 18002 times 10). The player drinks the mixture and dreams (startDream). She asks what the player saw. Branches: "big green mushrooms" (she says the player is the wrong one); [COPYRIGHT] (she thinks the player mocks her); [COPYRIGHT] (she accepts).
**Reward**: a charm that protects against the black world (item 300020).

### C30. Ville's sabotage contract (side)

**Giver**: Ville Sköldgangster [30], [148], [492]-[504] (objectives svamppatrullen1_3, svamppatrullen1_reward).
**Goal**: sabotage the Svamppatrullen gang's boss, who is in jail.
**Steps**:
1. Ville asks the player to scout the rival gang's base on the top floor of Deekula C (Locate the gang [494]; Scout the location [500]).
2. "Steal the bottle" [502]: take the Socca Junior bottle meant as a gift for the boss.
3. [COPYRIGHT] [496]: Ville gives a recipe; you need a chemical lab, Evil Bear candy bags from O-Market (a laxative), a mushroom extract and a syringe. Poison the bottle, swap it with the bottle in the gift box, and leave [504].
4. "Report your findings" [498]: report to Ville, who pays (1280 money in one file) and says the player would make a fine henchman.
**Branches**: refuse ([COPYRIGHT]); accept; steal the gift; refuse to kill ([COPYRIGHT]). Ville says the poison causes diarrhoea and a high, not death.
**Consequences**: the boss is later said to be addicted and sent to quarantine.

### C31. The speakeasy door and Malone's assistant (side, linking)

**Gate**: Mr. Door Phone [95] (SpeakeasyDoorMrDoor, SpeakeasyDoorUnlocked, MalonesAssistant).
**Steps**: claim to be Malone's assistant at the intercom. The door opens. The speakeasy and its secret room are open after that.

### C32. Passmore and the police (side, part of A4)

Covered under A4 step 8.

### C33. Colton's hole and the guards (side)

Covered under C9.

### C34. Kolhola's pods and the hostel (side)

**Giver**: Arvid Vinter [80] (unused in the file). The Warehom Hostel sells pods by night, with pods assigned by number.

### C35. Smuggling and drone delivery (side)

- **Pertti Korhonen's smuggling** [352]: 150 RM to get into the industrial zone (once the player has the smuggler flag, BazaarSmugglersKnowPlayer); 1000 RM for a van into the quarantine zone during the water plant's weekly downtime (Sunday 6 to 7; the trip back must be between 8 and 16, or the player is in quarantine for a week). The script removes 100 RM rather than 1000 RM, as written (possibly a bug).
- **Jane Miller's drone** [403]: a drone delivery into the quarantine area; 500 RM now, 2000 RM on delivery. Drones are disabled by the enemy. The Deekula B rooftop is suggested as the launch point.

### C36. The Redemption key and the Bazaar door (side, part of C6)

Covered under C6.

### C37. Erto's Murju pig and the furniture trade (side)

Covered under A11.

### C38. Card games (side)

Crosseven (a Finnish Sevens variant) and Shittypants (a chaotic card game). Welhö's game is part of A11. The bottom slum computer offers a login with the player's first name as a username; no password is shown to succeed. Winners and losers get lines.

### C39. Wasteland photography and red rocks (side)

**Giver**: the speaker in conv [180] (Wasteland photography).
**Goal**: collect red rocks in a field while a device is set up. The speaker asks for the player's last name.

### C40. Shadows and the Wasteland ghosts (side)

Several ghosts can be met after the player enters the Wasteland. The shed woman [91] (murdered, with a surgery scar from pregnancy), Shadow Mike [76] (asks for the tunnel key and the letter), the shadow man Jacob [69] (asks the player to find Caroline), Shadow Lagerfeld 1-3 [406]-[408] (a best friend, a wife, a warning), the burned tenant [413] (a fire set while he slept), the tunnel shadow [118] (silent), and the shadow priest [79]. Their roles in the plot are unclear.

### C41. Jacob, Caroline, Mike and Emilie (side, with the main mystery)

**Givers**: Jacob Cooper [69]; Mike [75] (MikeStart, MikeDaughter, MikeKnowsDaughter); Shadow Mike [76].
**Goal**: find Mike's daughter Emilie, and deliver a letter; find Jacob's daughter Caroline and tell her Jacob misses her.
**Steps** (tasks [506]-[518]):
1. Jacob asks the player to find his daughter Caroline and tell her he misses her [506, 508].
2. Mike, in the Bergmann tunnels, asks the player to give a letter to his daughter Emilie [518]. He does not know if she is alive.
3. [COPYRIGHT] [514]: Mike says Caroline might know where Emilie is. Caroline may be Jacob's daughter (unclear).
4. Mike sends the player to the Bazaar bookstore for the book [COPYRIGHT] [75:65].
5. Shadow Mike asks the player to take his key (B2) and the letter, and to find Emilie. Once the player agrees, MikeDaughter is set.
6. Emilie Palmer is listed as Upcoming in one file. The sources do not show her.

### C42. The wrong Passmore (side, mis-route)

**Giver**: the other Passmore in Tenement B [145].
The player who knocks on the wrong door is sent to Eliana by mistake. Eliana says the other Passmore still owes her.

### C43. The sleeping events (side, flavour and risk)

When the player sleeps, a random event may happen: money, OC, items or junk taken; an SMV infection; a beating; a police check that arrests the player if stolen goods are found; a visit from the Crazy Neighbor; or waking up for no reason.

### C44. Puzzle invitation (side, unclear)

**Giver**: Emilie Palmer (Upcoming) [61]. Offers a free puzzle: listen to a sound, go to the second-floor apartment, open the door with two question marks, and take the prize. The prize is a Mike question (unclear). Whether it is a trap is unclear.

### C45. Metro quiz (side)

**Giver**: Mason Jokinen [182]. Asks about the metro date, S.N.W, and the island's name. Trivia and history (see 05-world-and-lore.md).

---

## Part D. Chains with the most loose ends

- The Bank of Bazaar (A4) and the casino (A5) are connected but the final owner of the casino is unclear.
- The Ines chain (A9) has the most contradictory outcomes. The Joel and Pia endings differ between sources.
- The Osrat gate (A6) does not say what lies beyond it.
- The Welho's Mark and the Suurwelho's trials (A11) are not explained beyond the seal.
- Malone's fate (A2, B7) is unclear.
- The tenement collapse (B4) is threatened but not shown as happening.

---

## Part E. Summary of the side and main flags

- **Main**: A1 (opening), A2 (renovation), A3 (passports), A4 (bank), A5 (casino), A6 (gate), A7 (red mushroom experiment), A8 (Pia and Alice), A9 (Ines), A10 (phone and red rocks), A11 (Welho), A12 (Crazy Neighbor), A13 (endings).
- **Mines**: B1 (repository), B2 (permit), B3 (claims), B4 (tenement tunnel), B5 (Toivo's permit), B6 (speed cameras), B7 (Seamus), B8 (factory and elevator), B9 (janitor).
- **Side**: C1 to C45.
