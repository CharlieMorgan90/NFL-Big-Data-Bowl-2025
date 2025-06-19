Feature Extraction:

**Feature Extraction for Late Defensive Movements, Nearest Defender at the Snap, and Through-Snap Movement** was relatively straightforward compared to the more advanced logic required for detecting **Defensive Reactions to Offensive Motion**. I’ll briefly summarize the first three before diving deeper into the last.

1. **Late Defensive Movements**: A player was flagged as making a late movement if they changed position within the final second before the snap, exceeding both a distance and velocity threshold. This captured last-second shifts or hesitation.
2. **Nearest Defender to Each Receiver at the Snap**: For each receiver at the exact snap frame, I calculated the closest defender by computing the sum of absolute differences in their x and y positions. The defender with the lowest total distance was marked as the nearest defender.
3. **Defensive Player Movement Through the Snap**: If a defender maintained a speed above 1 across a 10-frame window (5 frames before and 5 after the snap), they were labeled as moving through the snap — a sign of late alignment or reactive shifting.

These features provided a foundation, but the real complexity and insight came from the fourth — **Defensive Reaction to Offensive Motion** — which required custom motion detection logic, vector analysis, and role classification in both man and zone coverage.

_4\._ **Defensive Reaction to Offensive Motion**

The process begins by identifying offensive players who go into motion, using the shiftSinceLineset and motionSinceLineset features provided in the dataset. However, I needed more precision — specifically, the exact **start and end frames** of motion. To accomplish this, I developed a custom motion detection function.

I first analyze the offensive player’s velocity across all pre-snap frames. Squaring the velocity reveals distinct patterns: motion begins with a sudden spike and ends with a sharp drop. But automating this insight is non-trivial. To avoid noise from huddle movements, I ignore the first third of frames and focus on the final two-thirds. Within that window, I find the frame with peak velocity, then loop backward and forward until the player’s velocity drops below a threshold, accurately identifying motion start and end. I also account for **double motion**, by scanning the remaining frames for any second peak that exceeds 50% of the original — if found, I record both motion windows.

Once the motion window is identified, the next challenge is determining **which defenders reacted**. I split this task into two approaches depending on coverage type (pff_manZone): **man** and **zone**.

In **man coverage**, only one defender should track the motioning offensive player. I calculate a **cosine similarity score** for each defender, comparing their directional velocity vectors to the offensive player’s during the motion window. To improve accuracy, I add two penalties: one for defenders whose distance to the offensive player fluctuates significantly, and another for defenders who barely moved (a **movement penalty**). The defender with the highest adjusted similarity score is selected as the one in coverage.

In **zone coverage**, multiple defenders may react. To capture this, I break the motion into **three equal segments** and compute similarity scores in each. This allows me to detect **coverage switches**, a hallmark of zone schemes — for example, if one defender tracks the first half of the motion, but another picks it up later. This multi-phase approach lets me analyze how responsibilities shift across time.

Using these methods, I created additional features for each play to quantify defensive reactions:

- segment_switch_count: Number of times defenders switched who was tracking the motion  

- num_defenders_react: Number of defenders that adjusted to the motion  

- avg_sim_score: Average movement similarity across defenders  

- max_sim_score: Maximum similarity score recorded  

- var_sim_score: Variance in defender similarity (to detect inconsistencies)  

- total_motion_duration: Total duration of defensive response movement before the snap  

These features proved valuable in modeling and understanding **pre-snap defensive confusion**, especially when paired with tracking-based metrics.
