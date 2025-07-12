Hey Splitgate devs,

First off, love the game and the work you’ve done so far on anti-cheat - it’s tough balancing fast-paced gameplay with fair play. That said, I wanted to suggest an idea that might help catch some of the more subtle cheats slipping through, especially the aim-assist and macro types that behave “human enough” to evade standard checks.

*Integrate client-side input behavior analysis to detect suspicious player actions by monitoring raw input patterns (mouse/controller) and comparing them against realistic human input curves.*


Why this could help:

> Many cheats nowadays are designed to mimic human input to avoid detection - perfect aim, perfect flicks, and frame-perfect triggerbotting.

>  Human input naturally contains noise, jitter, and irregular timing; bots or macros tend to have unnaturally smooth or mechanically perfect input curves.

> Monitoring “impossible state transitions” - like instantly flicking 180 degrees with pixel-perfect accuracy or firing within milliseconds of unscoping - could flag suspicious behavior in real-time.


How it could work in practice:

> Log anonymized input deltas client-side (mouse movement per frame, keypress durations, controller stick angles).

> Analyze input curves for smoothness, acceleration patterns, jitter, and timing irregularities.

> Detect impossible reaction times or input sequences that no human could realistically perform.

> Flag suspicious data and send lightweight summaries to servers for further validation.

> Optionally integrate lightweight ML models trained on legit player input to classify behavior.

Some considerations:

> Privacy is key - no raw input data should be stored long-term, and data should be anonymized.

> This wouldn’t be a silver bullet but an additional detection layer alongside server validation and behavioral heuristics.

> False positives can be minimized by combining this data with other game state info and reports.

> Could also provide insights for banning or soft-banning repeat offenders without ruining legit player experience.


I’d love to hear your thoughts on this, and if you think it’s something worth exploring. I believe this method could make it significantly harder for cheaters to fly under the radar, especially in a game as fast and precise as Splitgate.

Thanks for reading, and keep up the awesome work!

Example:              

// Store recent input reports for each player (could be in memory or a database)
const playerInputData = new Map();

// Thresholds - these would be tuned with real gameplay data
const THRESHOLDS = {
  variance: 0.001,            // Very low variance = suspiciously uniform input
  smoothness: 0.95,           // Extremely smooth inputs are suspicious
  impossibleFlicks: 1         // Number of impossible flicks tolerated before flagging
};

function receiveInputReport(report) {
  const { playerId, inputVariance, inputSmoothness, impossibleFlickTimestamps } = report;

  if (!playerInputData.has(playerId)) {
    playerInputData.set(playerId, []);
  }

  // Add the new report to this player's history
  const reports = playerInputData.get(playerId);
  reports.push(report);

  // Keep only the last 5 reports (adjust window size as needed)
  if (reports.length > 5) {
    reports.shift();
  }

  // Calculate averages and aggregates over the recent reports
  const avgVariance = reports.reduce((sum, r) => sum + r.inputVariance, 0) / reports.length;
  const avgSmoothness = reports.reduce((sum, r) => sum + r.inputSmoothness, 0) / reports.length;

  // Count all flicks across recent reports
  const totalImpossibleFlicks = reports.reduce(
    (count, r) => count + (r.impossibleFlickTimestamps ? r.impossibleFlickTimestamps.length : 0),
    0
  );

  // Check if this player shows suspicious input behavior consistently
  if (
    avgVariance < THRESHOLDS.variance &&
    avgSmoothness > THRESHOLDS.smoothness &&
    totalImpossibleFlicks > THRESHOLDS.impossibleFlicks
  ) {
    flagPlayer(playerId);
  }
}

function flagPlayer(playerId) {
  console.warn(`Player ${playerId} flagged for suspicious input behavior.`);
  // TODO: implement ban, softban, or monitoring here
}
