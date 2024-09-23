<!-- Thomas McCoy -->

<script>
  let time = new Date();
  let updateTime = () => time = new Date();
  setInterval(updateTime, 1000);

  let daysTracked = 0;

  let newEntryHoursSlept = 0;
  let newEntryWokenUp = "Yes";
  let newEntryTimesWokenUp = 0;
  let newEntryMorningsFeelings;
  let newEntryJournal;
  let newEntryScreenTime = "Yes";
  var entries = [];

  let editEntry = false;

  let sleepHoursGoal = 8;
  let wokenUpGoal = 2;
  let screenTimeGoal = 0;
  let averageHoursSlept = 0;
  let totalNumberOfTimesWokenUp = 0;
  let totalNumberOfDaysWithScreenTime = 0;

  let trackMorningFeelings = true;
  let trackMorningFeelingsButtonLabel = "Remove Activity";

  let trackJournal = true;
  let trackJournalButtonLabel = "Remove Activity";

  let editEntriesButtonLabel = "Edit Entries";

  $: if (newEntryHoursSlept < 0) newEntryHoursSlept = 0;
  $: if (newEntryHoursSlept > 24) newEntryHoursSlept = 24;
  $: if (typeof newEntryHoursSlept != "number") newEntryHoursSlept = 0;
  $: if (newEntryWokenUp === "No") newEntryTimesWokenUp = 0;
  $: if (typeof newEntryTimesWokenUp != "number") newEntryTimesWokenUp = 0;
  $: if (sleepHoursGoal < 0) sleepHoursGoal = 0;
  $: if (sleepHoursGoal > 24) sleepHoursGoal = 24;
  $: if (typeof sleepHoursGoal != "number") sleepHoursGoal = 0;
  $: if (wokenUpGoal < 0) wokenUpGoal = 0;
  $: if (typeof wokenUpGoal != "number") wokenUpGoal = 0;
  $: if (screenTimeGoal < 0) screenTimeGoal = 0;
  $: if (screenTimeGoal > 7) screenTimeGoal = 7;
  $: if (typeof screenTimeGoal != "number") screenTimeGoal = 0;
  $: {
    for (let i = 0; i < entries.length; i++) {
      if (entries[i].hoursSlept < 0) {
        entries[i].hoursSlept = 0;
      }
      if (entries[i].hoursSlept > 24) {
        entries[i].hoursSlept = 24;
      }
      if (typeof entries[i].hoursSlept != "number") entries[i].hoursSlept = 0;
      if (entries[i].wokenUp === "No") {
        entries[i].timesWokenUp = 0;
      }
      if (entries[i].timesWokenUp < 0) {
        entries[i].timesWokenUp = 0;
      }
      if (typeof entries[i].timesWokenUp != "number") entries[i].timesWokenUp = 0;
    }
  }
  $: { 
    let daysToComputeAverage = 0;
    if (daysTracked >= 7) {
      daysToComputeAverage = 7;
    }
    else {
      daysToComputeAverage = daysTracked;
    }
    if (daysTracked > 0) {
      let totalHoursSlept = 0;
      totalNumberOfTimesWokenUp = 0;
      totalNumberOfDaysWithScreenTime = 0;
      let j = daysTracked - daysToComputeAverage;
      for (let k = entries.length-1; k >= j; k--) {
        totalHoursSlept += entries[k].hoursSlept;
        totalNumberOfTimesWokenUp += entries[k].timesWokenUp;
        if (entries[k].screenTime === "Yes") {
          totalNumberOfDaysWithScreenTime++;
        }
      }
      averageHoursSlept = totalHoursSlept / daysToComputeAverage;
    }
  }
  
  // This array holds a value for each goal tracked
  // For index 0, 0 means you're far from your sleep goal, 1 means you're close, and 2 means you're meeting it exactly
  // For index 1 and 2, 0 means you're not meeting your times woken up goal and screen time goals respectively
  // For index 1 and 2, 1 means you're meeting your times woken up goal and screen time goals respectively
  let goalsMet = [0, 1, 1];
  let averageSleepMessage;
  $: {
    if ((averageHoursSlept - sleepHoursGoal) > 0.25) {
      averageSleepMessage = "You're getting too much sleep! Average of " + Math.abs(averageHoursSlept - sleepHoursGoal) + " hours away from your goal.";
      goalsMet[0] = 0;
    }
    else if ((averageHoursSlept - sleepHoursGoal) < -0.25) {
      averageSleepMessage = "You're not getting enough sleep! Average of " + Math.abs(averageHoursSlept - sleepHoursGoal) + " hours away from your goal.";
      goalsMet[0] = 0;
    }
    else if (Math.abs(averageHoursSlept - sleepHoursGoal) < 0.25 && Math.abs(averageHoursSlept - sleepHoursGoal) != 0){
      averageSleepMessage = "You're just about meeting your goal! Average of " + Math.abs(averageHoursSlept - sleepHoursGoal) + " hours away from your goal.";
      goalsMet[0] = 1;
    }
    else {
      averageSleepMessage = "You're meeting your goal exactly!";
      goalsMet[0] = 2;
    }
  }

  let timesWokenUpMessage;
  $: {
    if (totalNumberOfTimesWokenUp > wokenUpGoal) {
      timesWokenUpMessage = "You're waking up more often that you'd like. You're " + (totalNumberOfTimesWokenUp - wokenUpGoal) + " past your goal.";
      goalsMet[1] = 0;
    }
    else {
      timesWokenUpMessage = "You're not waking up more often than you'd like!";
      goalsMet[1] = 1;
    }
  }

  let screenTimeMessage;
  $: {
    if (totalNumberOfDaysWithScreenTime > screenTimeGoal) {
      screenTimeMessage = "You're having too much screen time an hour before bed. You're " + (totalNumberOfDaysWithScreenTime - screenTimeGoal) + " past your goal.";
      goalsMet[2] = 0;
    }
    else {
      screenTimeMessage = "You're not having too much screen time an hour before bed!";
      goalsMet[2] = 1;
    }
  }

  // An svg will be displayed based on how many goals are met
  let sumGoals = 2;
  $: { 
    sumGoals = 0;
    for (let z = 0; z < 3; z++) {
      sumGoals += goalsMet[z];
    }
  }
  
  function addEntry() {
    if (newEntryMorningsFeelings == undefined) {
      newEntryMorningsFeelings = "No particular feelings";
    }
    else {
      if (newEntryMorningsFeelings.length == 0) {
        newEntryMorningsFeelings = "No particular feelings";
      }
    }
    if (newEntryJournal == undefined) {
      newEntryJournal = "No journal written";
    }
    if (trackMorningFeelings == false) {
      newEntryMorningsFeelings = null;
    }
    if (trackJournal == false) {
      newEntryJournal = null;
    }
    let dateAsString = time.toDateString();
    entries.push({date: dateAsString, hoursSlept: newEntryHoursSlept, wokenUp: newEntryWokenUp, timesWokenUp: newEntryTimesWokenUp, morningFeelings: newEntryMorningsFeelings, journal: newEntryJournal, screenTime: newEntryScreenTime});
    daysTracked++;
    alert("You have saved the entry.");
  }

  function editEntries() {
    editEntry = !editEntry;
    if (editEntry == true) {
      alert("You can now edit your entries.");
      editEntriesButtonLabel = "Stop Editing Entries";
    }
    else {
      alert("You have exited the edit mode.");
      editEntriesButtonLabel = "Edit Entries";
      for(let g = 0; g < entries.length; g++) {
        if (entries[g].morningFeelings != null) {
          if (entries[g].morningFeelings.length == 0) {
            entries[g].morningFeelings = "No particular feelings";
          }
        }
      }
    }
  }

  function reverseTrackMorningFeelings() {
    trackMorningFeelings = !trackMorningFeelings;
    if (trackMorningFeelings == true) {
      trackMorningFeelingsButtonLabel = "Remove Activity";
      alert("You are now tracking your morning feelings.");
    }
    else {
      trackMorningFeelingsButtonLabel = "Track Activity";
      alert("You aren't tracking your morning feelings anymore.");
    }
  }

  function reverseTrackJournal() {
    trackJournal = !trackJournal;
    if (trackJournal == true) {
      trackJournalButtonLabel = "Remove Activity";
      alert("You are now tracking your journal.");
    }
    else {
      trackJournalButtonLabel = "Track Activity";
      alert("You aren't tracking your journal anymore.");
    }
  }
</script>


<body>
  <div id="generalInfo">
    <br/>
    <h1>Thomas McCoy</h1>
    <br/>
    <p>{time}</p>
    <br/>
    <p>Number of days tracked: {daysTracked}</p>
  </div>
  <div id=goalResultsSection>
    <h1>Goal Results</h1>
    <svg width="400" height="90">
      {#if sumGoals == 4}
        <polygon points="200, 5 170, 90 245, 37.5 155, 37.5 230, 90" fill="yellow"stroke-width="1"/>
      {:else if sumGoals > 0}
        <polygon points="225, 5 235, 10 200, 60 175, 40 185, 30 200, 50" fill="green"stroke-width="1"/>
      {:else}
        <polygon points="175, 10 185, 5 215, 30 245, 5 255, 10 225, 30 255, 55 245, 60 215, 35 185, 60 175, 55 205, 30" fill="red"stroke-width="1"/>
      {/if}
    </svg>
    <p>Average hours of sleep (past week): {averageHoursSlept}</p>
    <p>{averageSleepMessage}</p>
    <br/>
    <p>Number of times woken up during the middle of the night (past week): {totalNumberOfTimesWokenUp}</p>
    <p>{timesWokenUpMessage}</p>
    <br/>
    <p>Number of days with screen time within an hour of going to bed (past week): {totalNumberOfDaysWithScreenTime}</p>
    <p>{screenTimeMessage}</p>
    <br/>
  </div>
  <div id=pastEntriesSection>
    <h1>Past entries</h1>
    <div>
      {#each {length: daysTracked} as entry, i}
        {#if editEntry == true}
          <p id=entryDate>{entries[i].date}</p>
          <p class=pastEntryData>Hours slept: </p>
          <input type="number" min="0" max ="24" bind:value={entries[i].hoursSlept}>
          <p class=pastEntryData>Were you woken up at all during the night?: </p>
          {#each ["Yes", "No"] as wokenUp}
          <label>
            <input
              type="radio"
              name="wokenUpEdit"
              value={wokenUp}
              bind:group={entries[i].wokenUp}
            />
            {wokenUp}&nbsp&nbsp&nbsp
          </label>
          {/each}
          <p class=pastEntryData>Times woken up: </p>
          <input disabled={entries[i].wokenUp === "No"} type="number" min="0" bind:value={entries[i].timesWokenUp}>
          <p class=pastEntryData>Did you have any screen time within an hour of going to bed? </p>
          {#each ["Yes", "No"] as screenTime}
          <label>
            <input
              type="radio"
              name="screenTimeEdit"
              value={screenTime}
              bind:group={entries[i].screenTime}
            />
            {screenTime}&nbsp&nbsp&nbsp
          </label>
          {/each}
          {#if entries[i].morningFeelings != null}
            <p class=pastEntryData>How did you feel in the morning? </p>
            {#each ["Refreshed", "Tired", "Happy", "Sad"] as morningFeelings}
            <label>
              <input
                type="checkbox"
                name="morningsFeelingsEdit"
                value={morningFeelings}
                bind:group={entries[i].morningFeelings}
              />
              {morningFeelings}&nbsp&nbsp&nbsp
            </label>
            {/each}
          {/if}
          {#if entries[i].journal != null}
            <p class=pastEntryData>Journal: </p>
            <input id="journalEntry" bind:value={entries[i].journal}>
          {/if}
          <br/>
        {:else}
          <p id=entryDate>{entries[i].date}</p>
          <p class=pastEntryData>Hours slept: {entries[i].hoursSlept}</p>
          <br/>
          <p class=pastEntryData>Were you woken up at all during the night?: {entries[i].wokenUp}</p>
          <br/>
          <p class=pastEntryData>Times woken up: {entries[i].timesWokenUp}</p>
          <br/>
          <p class=pastEntryData>Did you have any screen time within an hour of going to bed? {entries[i].screenTime}</p>
          <br/>
          {#if entries[i].morningFeelings != null}
            <p class=pastEntryData>How did you feel in the morning? {entries[i].morningFeelings}</p>
            <br/>
          {/if}
          {#if entries[i].journal != null}
            <p class=pastEntryData>Journal: {entries[i].journal}</p>
            <br/>
          {/if}
          <br/>
        {/if}
      {/each}
      <button id="editEntries" on:click={editEntries}>{editEntriesButtonLabel}</button>
    </div>
  </div>
  <div id=goalsSection>
    <h1>Goals</h1>
    <label class=goalLabels>
      Hours of sleep per night?
      <input type="number" min="0" max="24" bind:value={sleepHoursGoal}/>
    </label>
    <br/>
    <label class=goalLabels>
      Number of times you woken up in the middle of the night (for a week)?
      <input type="number" min="0" bind:value={wokenUpGoal}/>
    </label>
    <br/>
    <label class=goalLabels>
      Nights with screen time within an hour of going to bed (for a week)?
      <input type="number" min="0" bind:value={screenTimeGoal}/>
    </label>
  </div>
  <div id="activitiesSection">
    <h1>Activities</h1>
      <label class="activityLabels">
        Hours slept: 
        <input type="number" min="0" max ="24" bind:value={newEntryHoursSlept}>
      </label>
      <br/>
      <label class="activityLabels">
        Did you wake up at all in the middle of the night (that you're aware of)?
        <br/>
        {#each ["Yes", "No"] as wokenUp}
          <label>
            <input
              type="radio"
              name="wokenUpEntry"
              value={wokenUp}
              bind:group={newEntryWokenUp}
            />
            {wokenUp}&nbsp&nbsp&nbsp
          </label>
        {/each}
      </label>
      <br/>
      <label class="activityLabels">
        How many times did you wake up?
        <input disabled={newEntryWokenUp === "No"} type="number" min="0" bind:value={newEntryTimesWokenUp}>
      </label>
      <br/>
      <label class="activityLabels">
        Did you have any screen time within an hour of going to bed?
        <br/>
        {#each ["Yes", "No"] as screenTime}
          <label>
            <input
              type="radio"
              name="screenTimeNewEntry"
              value={screenTime}
              bind:group={newEntryScreenTime}
            />
            {screenTime}&nbsp&nbsp&nbsp
          </label>
        {/each}
      </label>
      <br/>
      <label class="activityLabels">
        How did you feel when you woke up?
        <br/>
        {#each ["Refreshed", "Tired", "Happy", "Sad"] as morningFeelings}
          <label>
            <input
              disabled={trackMorningFeelings == false}
              type="checkbox"
              name="morningsFeelingsNewEntry"
              value={morningFeelings}
              bind:group={newEntryMorningsFeelings}
            />
            {morningFeelings}&nbsp&nbsp&nbsp
          </label>
        {/each}
      </label>
      <button class="removalButton" on:click={reverseTrackMorningFeelings}>{trackMorningFeelingsButtonLabel}</button>
      <br/>
      <br/>
      <label class="activityLabels">
        Write an entry about your feelings through the day.
        <br/>
        <input disabled={trackJournal == false} id="journalEntry" bind:value={newEntryJournal}>
      </label>
      <button class="removalButton" on:click={reverseTrackJournal}>{trackJournalButtonLabel}</button>
      <br/>
      <br/>
    <button id="submitEntry" on:click={addEntry}>Submit Entry</button>
  </div>
</body>