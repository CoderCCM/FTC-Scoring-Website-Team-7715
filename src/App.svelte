<style>
	td, tr th {
		border: 1px solid black;
		text-align: center;
		padding: 2px;
	}
</style>
<script>
var ffParams = [
		{ name: 'Duck Delivery', points: 10, type:"bool", pointsAwarded: false, group: "Autonomous"},
		{ name: 'Freight In Storage', points: 2, type:"infinite", count: 0, group: "Autonomous", linkToId: "ts"},
		{ name: 'Freight In Hub L1', points: 6, type:"infinite", count: 0, group: "Autonomous", linkToId: "t1"},
		{ name: 'Freight In Hub L2', points: 6, type:"infinite", count: 0, group: "Autonomous", linkToId: "t2"},
		{ name: 'Freight In Hub L3', points: 6, type:"infinite", count: 0, group: "Autonomous", linkToId: "t3"},
		{ name: 'Freight Bonus 1', type:"choice", choices:{"Team": 20, "Duck": 10, "None": 0}, choice: "None", group: "Autonomous"},
        { name: 'Freight Bonus 2', type:"choice", choices:{"Team": 20, "Duck": 10, "None": 0}, choice: "None", group: "Autonomous"},
		{ name: 'Parking 1 Storage Unit', type:"choice", choices:{"Full": 6, "Partly": 3, "None": 0}, choice: "None", group: "Autonomous"},
		{ name: 'Parking 1 Warehouse', type:"choice", choices:{"Full": 10, "Partly": 5, "None": 0}, choice: "None", group: "Autonomous"},
		{ name: 'Parking 2 Storage Unit', type:"choice", choices:{"Full": 6, "Partly": 3, "None": 0}, choice: "None", group: "Autonomous"},
		{ name: 'Parking 2 Warehouse', type:"choice", choices:{"Full": 10, "Partly": 5, "None": 0}, choice: "None", group: "Autonomous"},
		
		
		
		{ name: 'Freight In Storage', points: 1, type:"infinite", count: 0, group: "TeleOp", id: "ts", linkCount: 0},
		{ name: 'Freight In Hub L1', points: 2, type:"infinite", count: 0, group: "TeleOp", id: "t1", linkCount: 0},
		{ name: 'Freight In Hub L2', points: 4, type:"infinite", count: 0, group: "TeleOp", id: "t2", linkCount: 0},
		{ name: 'Freight In Hub L3', points: 6, type:"infinite", count: 0, group: "TeleOp", id: "t3", linkCount: 0},
		{ name: 'Freight In Shared', points: 4, type:"infinite", count: 0, group: "TeleOp"},
		
		
		
		{ name: 'Ducks Delivered', points: 6, type:"infinite", count: 0, group: "End Game"},
		{ name: 'Alliance Shipping Hub', type:"choice", choices:{"Balanced": 10, "Leaning": 0}, choice: "Leaning", group: "End Game"},
		{ name: 'Shared Shipping Hub', type:"choice", choices:{"Leaning": 20, "Balanced": 0}, choice: "Balanced", group: "End Game"},
		{ name: 'Parking 1', type:"choice", choices:{"Full": 6, "Partly": 3, "None": 0}, choice: "None", group: "End Game"},
		{ name: 'Parking 2', type:"choice", choices:{"Full": 6, "Partly": 3, "None": 0}, choice: "None", group: "End Game"},
		{ name: 'Capping', type:"choice", choices:{"Two": 30, "One": 15, "Zero": 0}, choice: "Zero", group: "End Game"}
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
        {param.count}
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