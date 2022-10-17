<style>
	td, tr th {
		border: 1px solid black;
		text-align: center;
		padding: 2px;
	}
</style>
<script>
var ffParams = [
		{ name: 'Parked in Terminal or Substation', points: 2, type:"bool", pointsAwarded: false, group: "Autonomous"},
		{ name: 'Cone in Terminal', points: 1, type:"infinite", pointsAwarded: false, group: "Autonomous", linkToId: "ts"},
		{ name: 'Ground', points: 2, type:"infinite", count: 0, group: "Autonomous", linkToId: "t1"},
		{ name: 'Low', points: 3, type:"infinite", count: 0, group: "Autonomous", linkToId: "t2"},
		{ name: 'Medium', points: 4, type:"infinite", count: 0, group: "Autonomous", linkToId: "t3"},
		{ name: 'High', points: 5, type:"infinite", count: 0, group: "Autonomous", linkToId: "t4"},
		{ name: 'Signal Zone Park', type:"choice", choices:{"Team Signal": 20, "Signal": 10, "None": 0}, choice: "None", group: "Autonomous"},
		
		
		{ name: 'Cone in Terminal', points: 1, type:"infinite", count: 0, group: "TeleOp", id: "ts", linkCount: 0},
		{ name: 'Ground', points: 2, type:"infinite", count: 0, group: "TeleOp", linkToId: "t1", linkCount: 0},
		{ name: 'Low', points: 3, type:"infinite", count: 0, group: "TeleOp", linkToId: "t2", linkCount: 0},
		{ name: 'Medium', points: 4, type:"infinite", count: 0, group: "TeleOp", linkToId: "t3", linkCount: 0},
		{ name: 'High', points: 5, type:"infinite", count: 0, group: "TeleOp", linkToId: "t4", linkCount: 0},
		
		
		{ name: 'Parked in Terminal', points: 2, type:"bool", pointsAwarded: false, group: "End Game"},
		{ name: 'Alliance Cone Own Junction', points: 3, type:"infinite", count: 0, group: "End Game"},
		{ name: 'Junction Owned by Beacon', type:"choice", choices:{"Yes": 10, "No": 0}, choice: "No", group: "End Game"},
		{ name: 'Completed Circuit', type:"choice", choices:{"Yes": 20, "No": 0}, choice: "No", group: "End Game"},
	];



var parameters;
if (localStorage.getItem("config")!=null) {
    parameters = JSON.parse(localStorage.getItem("config"));
} else {
    parameters = [...ffParams];
}




	var groups = [];
	var groupPoints = []
	

		parameters.forEach(function(item) {
			if (groups.indexOf(item.group)==-1) {
				groups.push(item.group);
				groupPoints.push(0)
			}
		})

	
	
	
	var totalPoints = 0;
	
	$:{
		totalPoints = 0;
		groupPoints.forEach(function(item, index) {
			groupPoints[index] = 0;
		});
		parameters.forEach(function(item) {
            if(item.linkToId) {
                var linkedParam = parameters.filter(function(param) {
                    return param.id && param.id == item.linkToId
                });
                linkedParam = linkedParam[0]
                parameters[parameters.indexOf(linkedParam)].linkCount = item.count;
            }

			if (item.type == "infinite") {
				totalPoints += (item.points*(item.count+(item.linkCount ? item.linkCount : 0)))
				groupPoints[groups.indexOf(item.group)] += (item.points*(item.count+(item.linkCount ? item.linkCount : 0)))
			}
			if (item.type == "bool") {
				if (item.pointsAwarded) {
					totalPoints += item.points
					groupPoints[groups.indexOf(item.group)] += item.points
				}
			}
			if (item.type == "choice") {
				totalPoints += item.choices[item.choice]
				groupPoints[groups.indexOf(item.group)] += item.choices[item.choice]
			}
		});
	}


    function updateFromTextarea() {
        parameters = JSON.parse(document.getElementById("config").value);
        localStorage.setItem("config", JSON.stringify(parameters));
    }
</script>

<h1>FTC Scorer ~ 17301</h1>
Config File: <textarea id="config" on:change={()=>updateFromTextarea()}>{JSON.stringify(parameters)}</textarea>
<button on:click={() => {parameters = ffParams; localStorage.setItem("config", JSON.stringify(parameters));}}>Revert to Freight Frenzy Config File</button>
<hr>


	{#each groups as group}
		<h2>
			{group} - {groupPoints[groups.indexOf(group)]}
	</h2>
	<table id={group}>
		<tr><th>Task</th><th>Scoring</th><th>Points</th></tr>
		{#each parameters as param}
		{#if param.group == group}
		{#if param.type == "infinite"}
	<tr>
		<td>{param.name}</td><td>
        <button style="background-color: transparent" on:click={()=> param.count++}>
		+
		</button>
        {(param.count+(param.linkCount ? param.linkCount : 0))}
        <button style="background-color: transparent" on:click={()=> param.count--}>
		-
		</button></td><td>{(param.points*(param.count+(param.linkCount ? param.linkCount : 0)))}</td>
	</tr>
		{/if}
	{#if param.type == "bool"}
	<tr>
		<td>{param.name}</td><td><button style="background-color:{param.pointsAwarded ? 'lightgreen' : 'transparent'}" on:click={()=> param.pointsAwarded=true}>
		Yes
		</button><button style="background-color:{!param.pointsAwarded ? 'lightgreen' : 'transparent'}" on:click={()=> param.pointsAwarded=false}>
		No
		</button></td> <td>{param.pointsAwarded ? param.points : 0}</td>
	</tr>
		{/if}
	{#if param.type == "choice"}
	<tr>
		<td>{param.name}</td> <td>
		{#each Object.keys(param.choices) as choice}
		<button style="background-color: {param.choice == choice ? 'lightgreen' : 'transparent'}" on:click={()=> param.choice = choice}>{choice}</button>
		{/each}</td><td>{param.choices[param.choice]}</td>
	</tr>
		{/if}
	{/if}
	{/each}
	</table>
<hr>
	{/each}
	
	
{#each groups as group}
<h4>
	 {group} - {groupPoints[groups.indexOf(group)]}
</h4>
{/each}
<h2>
	Total Points : {totalPoints}
</h2>