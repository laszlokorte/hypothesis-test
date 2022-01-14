<script>
	let mean = [-.2,.3]
	let variance = [0.1,0.2]
	let sampleCount = 1000
	let prior = 0.5
	let boundary = 0
	let boundaryNoCost = 0
	let autoOptimize = false
	let stacked = false
	
	let maxCost = 2
	let cost = [0,1,1,0]
	
	let colors = ['blue', 'red']
	let costColors = ['#f5f', '#fa0','#0af','#0a0']

	$: swapped = mean[0] > mean[1] ? 1 : 0


	function pdf(mean, variance, x) {
		if(variance == 0) {
			return Math.abs(x-mean) < 2/sampleCount ? Math.sqrt(sampleCount) : 0
		} else {
			return 1/(variance*Math.pow(2*Math.PI, 2)) * Math.exp(-0.5*Math.pow(x-mean, 2) / Math.pow(variance, 2))
		}
	}

	function cdf(mean, variance, x) {
		if(variance == 0) {
			return x-mean > -1/sampleCount ? 1 : 0
		} else {
			return 0.5 * (1 + erf((x-mean)/Math.sqrt(2)/variance))
		}
	}

	function erf(x) {
		const enxx = Math.exp(-x*x)
		const sqpi = Math.sqrt(Math.PI)

		return 2/sqpi * Math.sign(x) * Math.sqrt(1-enxx) * (sqpi/2 + 31/200 * enxx - 341/8000 * Math.sqrt(1-enxx*enxx))
	}
	
	$: sampleCost = (p,x) => {
		const swap = mean[0] > mean[1] ? 1 : 0
		return x > boundary ? cost[2*p+ 1 - swap] : cost[2*p + swap]
	}
	
	const X = Array(sampleCount).fill(null).map((_,i) => 2*i/sampleCount - 1)
	
	$: samples = [
		X.map((x) => [x, prior * pdf(mean[0], variance[0], x)]),
		X.map((x) => [x, (1-prior) * pdf(mean[1], variance[1], x)])
	]

	$: integrations = [
		X.map((x) => [x, cdf(mean[0], variance[0], x)]),
		X.map((x) => [x, cdf(mean[1], variance[1], x)])
	]
	
	$: costs = [
		X.map((x) => sampleCost(0, x)),
		X.map((x) => sampleCost(1, x))
	]
	
	function optimizBoundary(prior, means, variances, cost) {
		const swap = means[0] > means[1] ? 1 : 0
		let sumLeft = 0
		let sumRight = ((prior)*cost[1-swap] + (1-prior)*cost[3-swap])*Math.sqrt(sampleCount)

		let bestSum = Infinity
		let bestI = 0

		for (var i = 0; i < sampleCount; i++) {
			sumLeft += (prior * pdf(mean[swap], variance[swap], X[i]))*cost[0+swap] 
			+ ((1-prior) * pdf(mean[1-swap], variance[1-swap], X[i]))*cost[2+swap]

			sumRight -= (prior * pdf(mean[swap], variance[swap], X[i]))*cost[1-swap] 
			+ ((1-prior) * pdf(mean[1-swap], variance[1-swap], X[i]))*cost[3-swap]

			
			if(sumLeft+sumRight < bestSum) {
				bestSum = sumLeft + sumRight
				bestI = i;
			}
		}

		return [X[bestI], bestSum]
	}

	$: if(autoOptimize) {
		const o = optimizBoundary(prior, mean, variance, cost)
		boundary = o[0]
	}

	const candidateCount = 50

	$: candidatePriors = Array(candidateCount).fill(null).map((_,i) => i/candidateCount).map(p => [p,(optimizBoundary(p, mean, variance, cost)[1])])

	$: bestPrior = candidatePriors.reduce(([p, highest], [pri, risk]) => (risk > highest ? [pri,risk] : [p, highest]))


	function flipY(y) {
		return 280 - y;
	}
</script>

<style>
	:global(body) {
		margin: 0;
		padding: 1em;
	}
	:global(*) {
		box-sizing: border-box;
	}
	dl {display: grid; grid-template-columns: max-content 1fr; align-items: center;gap:1em; margin: 0;grid-auto-columns: min-content; grid-auto-flow: column; padding: 0;justify-items: stretch;}
	input[type=range] {margin: 0; width: 100%; max-width: 100%; padding: 0;}
	input {margin: 0;}
	dd {margin: 0}
	dt {grid-column-start: 1;}
	label[for]::after {content:':';}
	
	.controls {
		width: 100%;
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(18em, 1fr));
		gap: 1em;
	}
	
	svg {
		max-width: 100%;
		width: 100%;
		height: auto;
		padding: 0;
		margin: 0
	}
	
	label {
		white-space: nowrap;
	}
</style>

<h1>
	Binary Hypothesis Test
</h1>

<div class="controls">
<fieldset>
	<legend>
		Hypothesis 1
	</legend>
<dl style={`color: ${colors[0]}`}>
	<dt><label for='mean_0'>H<sub>1</sub> Mean</label></dt>
	<dd><input id="mean_0" type="range" bind:value={mean[0]} min="-1" max="1" step="0.01" /></dd>
	<dt><label for='variance_0'>H<sub>1</sub> Variance</label></dt>
	<dd><input id="variance_0" type="range" bind:value={variance[0]} min="0" max="1" step="0.001" /></dd>
</dl>	
</fieldset>

<fieldset>
	<legend>
		Hypothesis 2
	</legend>
	<dl style={`color: ${colors[1]}`}>
	<dt><label for='mean_1'>H<sub>2</sub> Mean</label></dt>
	<dd><input id="mean_1" type="range" bind:value={mean[1]} min="-1" max="1" step="0.01" /></dd>
	<dt><label for='variance_1'>H<sub>2</sub> Variance</label></dt>
	<dd><input id="variance_1" type="range" bind:value={variance[1]} min="0" max="1" step="0.001" /></dd>
</dl>
</fieldset>
	

</div>
<fieldset>
	<legend>Prior</legend>
	<dl>
	<dt><label for='prior'>Prior</label></dt>
	<dd><input id="prior" type="range" bind:value={prior} min="0" max="1" step="0.01" /></dd>
</dl>
</fieldset>


	<fieldset>
	<legend>
		Costs
	</legend>
		<div class="controls">
			<dl>
	<dt><label for='cost_0' style={`color: ${costColors[0]}`}>Cost<sub>1|1</sub></label></dt>
	<dd><input id="cost_0" type="range" bind:value={cost[0]} min="0" max={maxCost} step="0.01" /></dd>
	<dt><label for='cost_1' style={`color: ${costColors[1]}`}>Cost<sub>2|1</sub></label></dt>
	<dd><input id="cost_1" type="range" bind:value={cost[1]} min="0" max={maxCost} step="0.01" /></dd>
	</dl>
		<dl>

	<dt><label for='cost_2'style={`color:${costColors[2]}`}>Cost<sub>1|2</sub></label></dt>
	<dd><input id="cost_2" type="range" bind:value={cost[2]} min="0" max={maxCost} step="0.01" /></dd>
	<dt><label for='cost_3' style={`color:${costColors[3]}`}>Cost<sub>2|2</sub></label></dt>
	<dd><input id="cost_3" type="range" bind:value={cost[3]} min="0" max={maxCost} step="0.01" /></dd>
</dl>
		</div>
	</fieldset>



<fieldset>
	<legend>
			<label><input type="checkbox" bind:checked={autoOptimize} /> Auto Optimize</label></legend>
	<dl>
		<dt><label for='boundary'>Decision Boundary</label></dt>
		<dd><input id="boundary" bind:value={boundary} type="range"  min="-1" max="1" step="0.01" disabled={autoOptimize} /></dd>
	</dl>
</fieldset>

<fieldset>
	<legend>
			View Options
	<label><input id="stacked" bind:checked={stacked} type="checkbox" /> Stack distributions</label>
</fieldset>


<svg viewBox="-20 -180 1080 700" stroke-width="2" font-size="24">
	<defs>
		<marker id="arrowhead" markerWidth="10" markerHeight="7" 
    refX="0" refY="3.5" orient="auto">
      <polygon points="0 0, 10 3.5, 0 7" />
    </marker>
	</defs>

	<g>
	<line x1="0" y1={flipY(0)} x2="1000" y2={flipY(0)} stroke="black" marker-end="url(#arrowhead)" />
	<line x1="0" y1={flipY(0)} x2="0" y2={flipY(400)} stroke="black" marker-end="url(#arrowhead)" />
	<line x1="0" y1={flipY(0)} x2="0" y2={flipY(-100)} stroke="black" marker-end="url(#arrowhead)" />
	<text x={25} y={flipY(400)} text-anchor="start">p(X)</text>
	<text x={25} y={flipY(-100)} text-anchor="start">Risk</text>
	<text x={1000} y={flipY(20)} text-anchor="start">X</text>


	{#if stacked}

	<path d={`M0,${flipY(0)} ` + samples[1].flatMap(([x,p],i) => ['L'+(500+500*x),flipY(2+(p+(stacked?samples[0][i][1]:0))*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[1]} opacity="1" />

	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p]) => ['L'+(500+500*x),flipY(2+p*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[0]} opacity="1" />
	
	{:else}

	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p]) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[0]} opacity="0.2" />

	<path d={`M0,${flipY(0)} ` + samples[1].flatMap(([x,p],i) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[1]} opacity="0.2" />


		
	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p]) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ')} stroke={colors[0]} fill="none" />
	<path d={`M0,${flipY(0)} ` + samples[1].flatMap(([x,p],i) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ')} stroke={colors[1]} fill="none" />
	{/if}
		
	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p],i) =>  x>=boundary ? [] : ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1] + costs[1][i]*samples[1][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={costColors[2+swapped]} />
		
	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p],i) => x>=boundary ? [] : ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={costColors[0+swapped]} />
	
		
	<path d={`M${boundary*500 + 500},${flipY(0)} ` + samples[0].flatMap(([x,p],i) =>  x<boundary ? [] : ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1] + costs[1][i]*samples[1][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={costColors[3-swapped]}  />
		
	<path d={`M${boundary*500 + 500},${flipY(0)} ` + samples[0].flatMap(([x,p],i) => x<boundary ? [] : [('L'+(500+500*x)),flipY(-(costs[0][i]*samples[0][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={costColors[1-swapped]}  />
	
		<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p],i) => [((x>boundary && i-1 && samples[0][i-1][0] < boundary) ? 'V' : 'L'+(500+500*x)),flipY(-(costs[0][i]*samples[0][i][1] + costs[1][i]*samples[1][i][1])*600)]).join(' ') + `V${flipY(0)}`} fill="none" stroke="black" stroke-width="1"  />
		
		
		
		<line x1={500+boundary*500} x2={500+boundary*500} y1={flipY(-100)} y2={flipY(300)} stroke="black" stroke-width="1" stroke-dasharray="15 5" />
	<text x={500+boundary*500} y={flipY(320)}   text-anchor="middle">Decision Boundary</text>
		
		<line x1={500+boundary*500} x2={500+boundary*500 - 50} y1={flipY(250)} y2={flipY(250)} stroke={mean[0] < mean[1] ? colors[0] : colors[1]}/>
		<line x1={500+boundary*500} x2={500+boundary*500 + 50} y1={flipY(250)} y2={flipY(250)} stroke={mean[1] < mean[0] ? colors[0] : colors[1]} />
		
		<polygon transform={`translate(${500+boundary*500+50} ${flipY(250)})`} fill={mean[1] < mean[0] ? colors[0] : colors[1]} points="0 -7, 20 0, 0 7" />
		<polygon transform={`translate(${500+boundary*500-50} ${flipY(250)})`} fill={mean[0] < mean[1] ? colors[0] : colors[1]} points="0 -7, -20 0, 0 7" />
		</g>
</svg>

<h2>
	Explanation
</h2>
		
	<p>
	The goal is to decide if a given sample <code>x</code> is more likely to originate from one distribution (hypothesis 1) or from another one (hypthesis 2).
</p>
<p>
	It is assumed that both possible distributions are known and that we know the prior probability of any sample to be from the one or the other distribution. For example if the prior is 0.5 in general both distributions are equally likely. If the prior is 0.1 samples from one distribution are assumed to be much more likely than from the other.
</p>

<p>
	There exist 4 possible outcomes: 
</p>
<ol>
	<li>H<sub>1</sub> is true and H<sub>1</sub> is detected</li>
	<li>H<sub>1</sub> is true but H<sub>2</sub> is detected</li>
	<li>H<sub>2</sub> is true but H<sub>1</sub> is detected</li>
	<li>H<sub>2</sub> is true and H<sub>2</sub> is detected</li>
</ol>
	
<p>
		Each case has an associated cost. Case 1 and case 4 typacally have a cost of 0 because they are desired. The other two cases represent false positive and false negative results. Their cost is assigned individually.
</p>
<p>
	The goal is to find a decision boundary that devides the sample space into two regions that classify all samples and minimize the expected total cost (risk).
</p>

<p style="font-weight: bold; padding: 0 0 2em 0;">
	Above you can model two distributions, assign cost values and then try to move the decision boundary so that the area under the risk function is minimized.
</p>

<h3>Unknown Prior</h3>

<p>
	If the prior is not known, one can assume the worst case and pick the prior that leads to the highest lowest risk:
</p>

<svg viewBox="-20 -180 1080 500" stroke-width="2" font-size="24">
	<line x1="0" y1={flipY(0)} x2="1000" y2={flipY(0)} stroke="black" marker-end="url(#arrowhead)" />
		<line x1="0" y1={flipY(0)} x2="0" y2={flipY(400)} stroke="black" marker-end="url(#arrowhead)" />
		<text x={25} y={flipY(400)} text-anchor="start">Lowest Risk for Prior</text>
		<text x={1000} y={flipY(20)} text-anchor="start">Prior</text>

		<path d={`M0,${flipY(0)} ` + candidatePriors.map(([pri,risk]) => `L${pri*1000},${flipY(50*(risk))}`).join(' ')} stroke-width="2" stroke="black" fill="none" />

		<circle cx={bestPrior[0]*1000} cy={flipY(50*bestPrior[1])} r="5" />
		<text x={bestPrior[0]*1000} y={flipY(50*bestPrior[1]+20)} text-anchor="middle">Worst Prior</text>

</svg>

<p>
	Which prior is worst does depend on both the hypthesis and the costs.
</p>

<fieldset>
	<legend>
		Costs
	</legend>
		<div class="controls">
			<dl>
	<dt><label for='cost_0_bottom' style={`color: ${costColors[0]}`}>Cost<sub>1|1</sub></label></dt>
	<dd><input id="cost_0_bottom" type="range" bind:value={cost[0]} min="0" max={maxCost} step="0.01" /></dd>
	<dt><label for='cost_1_bottom' style={`color: ${costColors[1]}`}>Cost<sub>2|1</sub></label></dt>
	<dd><input id="cost_1_bottom" type="range" bind:value={cost[1]} min="0" max={maxCost} step="0.01" /></dd>
	</dl>
		<dl>

	<dt><label for='cost_2_bottom'style={`color:${costColors[2]}`}>Cost<sub>1|2</sub></label></dt>
	<dd><input id="cost_2_bottom" type="range" bind:value={cost[2]} min="0" max={maxCost} step="0.01" /></dd>
	<dt><label for='cost_3_bottom' style={`color:${costColors[3]}`}>Cost<sub>2|2</sub></label></dt>
	<dd><input id="cost_3_bottom" type="range" bind:value={cost[3]} min="0" max={maxCost} step="0.01" /></dd>
</dl>
		</div>
	</fieldset>

<h3>Unknown Costs</h3>

<p>If both the costs and the prior is unknown its still possible to find an optimal decision boundary by looking at the resulting ratio between <strong>true positives</strong> and <strong>false positives</strong> for the two given distributions. The resulting curve is called the <a href="https://en.wikipedia.org/wiki/Receiver_operating_characteristic">receiver operating characteristic curve</a>.</p>

<div>

	<svg viewBox="-350 -150 1600 1250" stroke-width="2" font-size="30">
			

	<line x1="0" y1={(0)} x2="1050" y2={(0)} stroke="black" marker-end="url(#arrowhead)" />
	<line x1="0" y1={(0)} x2="0" y2={(1050)} stroke="black" marker-end="url(#arrowhead)" />

	<text font-weight="bold" fill={colors[1]}  x={450 + 500 * boundaryNoCost} y={(-65)} text-anchor="end"><tspan dx="0" dy="0">false</tspan> <tspan dx="-110" dy="40">positives</tspan></text>

	<text fill={colors[1]} x={550 + 500 * boundaryNoCost} y={(-65)} text-anchor="start"><tspan dy="0">true</tspan> <tspan  dx="-110" dy="40">negatives</tspan></text>


	<text font-weight="bold" fill={colors[0]}  y={450 + 500 * boundaryNoCost} x={(-65)} text-anchor="end"><tspan dx="-0" dy="0">true</tspan> <tspan dx="-110" dy="40">positives</tspan></text>

	<text fill={colors[0]} y={550 + 500 * boundaryNoCost} x={(-65)} text-anchor="end"><tspan dy="0">false</tspan> <tspan  dx="-110" dy="40">negatives</tspan></text>

		<path d={integrations[0].flatMap(([x,p], i) => [
		(i>0?'L':'M')+
		(1000*integrations[0][i][1]),
		(1000*integrations[1][i][1])
		]).join(' ')
	} fill="none" stroke="black" />

	<circle cx={1000 * cdf(mean[0], variance[0], boundaryNoCost)} cy={1000 * cdf(mean[1], variance[1], boundaryNoCost)} r="15" />

	<circle cx={0} cy={500 + 500*boundaryNoCost} r="10" />
	<circle cx={500 + 500*boundaryNoCost} cy={0} r="10" />

<!-- 	<path d={`M0,${flipY(0)} ` + integrations[0].flatMap(([x,p]) => ['L'+(500+500*x),flipY((p)*600)]).join(' ')} fill="none" stroke={colors[0]} opacity="0.21" />

	<path d={`M0,${flipY(0)} ` + integrations[1].flatMap(([x,p],i) => ['L'+(500+500*x),flipY((1-p)*600)]).join(' ')} fill="none" stroke={colors[1]} opacity="0.1" /> -->

	<line x1="0" y1="0" x2="1000" y2="1000" stroke-dasharray="20 20" stroke="gray" />

	<path d={`M0,-5 ` + samples[1].flatMap(([x,p]) => ['L'+(500+500*x),-(5+p*600)]).join(' ') + `V${(-5)} Z`} fill={colors[1]} opacity="0.2" />

	<path d={`M-5,0 ` + samples[0].flatMap(([x,p],i) => ['L'+-(5+p*600),(500+500*x)]).join(' ') + `H${(-5)} Z`} fill={colors[0]} opacity="0.2" />
		
	<path d={`M0,-5 ` + samples[1].flatMap(([x,p]) => x>boundaryNoCost ? [] : ['L'+(500+500*x),-(5+p*600)]).join(' ') + `V${(-5)} Z`} fill={colors[1]} opacity="0.5" />

	<path d={`M-5,0 ` + samples[0].flatMap(([x,p]) => x>boundaryNoCost ? [] : ['L'+-(5+p*600),(500+500*x)]).join(' ') + `H${(-5)} Z`} fill={colors[0]} opacity="0.5" />
	
	</svg>

<div>
	
<fieldset>
	<legend>
		Hypothesis 1
	</legend>
<dl style={`color: ${colors[0]}`}>
	<dt><label for='mean_0'>H<sub>1</sub> Mean</label></dt>
	<dd><input id="mean_0" type="range" bind:value={mean[0]} min="-1" max="1" step="0.01" /></dd>
	<dt><label for='variance_0'>H<sub>1</sub> Variance</label></dt>
	<dd><input id="variance_0" type="range" bind:value={variance[0]} min="0" max="1" step="0.001" /></dd>
</dl>	
</fieldset>

<fieldset>
	<legend>
		Hypothesis 2
	</legend>
	<dl style={`color: ${colors[1]}`}>
	<dt><label for='mean_1'>H<sub>2</sub> Mean</label></dt>
	<dd><input id="mean_1" type="range" bind:value={mean[1]} min="-1" max="1" step="0.01" /></dd>
	<dt><label for='variance_1'>H<sub>2</sub> Variance</label></dt>
	<dd><input id="variance_1" type="range" bind:value={variance[1]} min="0" max="1" step="0.001" /></dd>
</dl>
</fieldset>
<fieldset>
	<legend>Without costs</legend>
	<dl>
		<dt><label for='boundary_bottom'>Decision Boundary</label></dt>
		<dd><input id="boundary_bottom" bind:value={boundaryNoCost} type="range"  min="-1" max="1" step="0.01" /></dd>
	</dl>
</fieldset>

</div>
	
</div>