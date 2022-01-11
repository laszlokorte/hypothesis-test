<script>
	let mean = [-.2,.3]
	let variance = [0.1,0.2]
	let sampleCount = 500
	let prior = 0.5
	let boundary = 0
	let autoOptimize = false
	
	let maxCost = 2
	let cost = [0,1,1,0]
	
	let colors = ['blue', 'red']
	
	$: pdf = (p, x) => {
		const factor = Math.abs(p - prior)
		if(variance[p] == 0) {
			return Math.abs(x-mean[p]) < 2/sampleCount ? factor*Math.sqrt(sampleCount) : 0
		} else {
			return factor * 1/(variance[p]*Math.pow(2*Math.PI, 2)) * Math.exp(-0.5*Math.pow(x-mean[p], 2) / Math.pow(variance[p], 2))
		}
	}
	
	$: sampleCost = (p,x) => {
		return x > boundary ? cost[2*p+ 1] : cost[2*p]
	}
	
	$: X = Array(sampleCount).fill(null).map((_,i) => 2*i/sampleCount - 1)
	
	$: samples = [
		X.map((x) => [x, pdf(0, x)]),
		X.map((x) => [x, pdf(1, x)])
	]
	
	$: costs = [
		X.map((x) => sampleCost(0, x)),
		X.map((x) => sampleCost(1, x))
	]
	
	function optimizBoundary() {
		
	}

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
<dl>
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
	<dl>
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
	<dt><label for='cost_0' style="color: #33a">Cost(t=H<sub>1</sub>|H<sub>1</sub>)</label></dt>
	<dd><input id="cost_0" type="range" bind:value={cost[0]} min="0" max={maxCost} step="0.01" /></dd>
	<dt><label for='cost_1' style="color: #33a">Cost(t=H<sub>2</sub>|H<sub>1</sub>)</label></dt>
	<dd><input id="cost_1" type="range" bind:value={cost[1]} min="0" max={maxCost} step="0.01" /></dd>
	</dl>
		<dl>

	<dt><label for='cost_2'style="color: #a33">Cost(t=H<sub>1</sub>|H<sub>2</sub>)</label></dt>
	<dd><input id="cost_2" type="range" bind:value={cost[2]} min="0" max={maxCost} step="0.01" /></dd>
	<dt><label for='cost_3' style="color: #a33">Cost(t=H<sub>2</sub>|H<sub>2</sub>)</label></dt>
	<dd><input id="cost_3" type="range" bind:value={cost[3]} min="0" max={maxCost} step="0.01" /></dd>
</dl>
		</div>
	</fieldset>



<fieldset>
	<legend>
			<label><input type="checkbox" bind:checked={autoOptimize} /> Auto Optimize (not implemented yet)</label></legend>
	<dl>
		<dt><label for='boundary'>Decision Boundary</label></dt>
		<dd><input id="boundary" type="range" bind:value={boundary} min="-1" max="1" step="0.01" disabled={autoOptimize} /></dd>
	</dl>
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
	
	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p]) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ') + 'V${flipY(0)} Z'} fill={colors[0]} opacity="0.2" />
	<path d={`M0,${flipY(0)} ` + samples[1].flatMap(([x,p]) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ') + 'V${flipY(0)} Z'} fill={colors[1]} opacity="0.2" />
		
	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p]) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ')} stroke={colors[0]} fill="none" />
	<path d={`M0,${flipY(0)} ` + samples[1].flatMap(([x,p]) => ['L'+(500+500*x),flipY(1+p*600)]).join(' ')} stroke={colors[1]} fill="none" />
		
		
	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p],i) =>  x>boundary ? [] : ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1] + costs[1][i]*samples[1][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[1]} />
		
	<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p],i) => x>boundary ? [] : ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[0]} />
	
		
	<path d={`M${boundary*500 + 500},${flipY(0)} ` + samples[0].flatMap(([x,p],i) =>  x<boundary ? [] : ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1] + costs[1][i]*samples[1][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[1]}  />
		
	<path d={`M${boundary*500 + 500},${flipY(0)} ` + samples[0].flatMap(([x,p],i) => x<boundary ? [] : ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill={colors[0]}  />
	
		<path d={`M0,${flipY(0)} ` + samples[0].flatMap(([x,p],i) => ['L'+(500+500*x),flipY(-(costs[0][i]*samples[0][i][1] + costs[1][i]*samples[1][i][1])*600)]).join(' ') + `V${flipY(0)} Z`} fill="none" stroke="black" stroke-width="1"  />
	
		
		
		<line x1={500+boundary*500} x2={500+boundary*500} y1={flipY(-100)} y2={flipY(300)} stroke="black" stroke-dasharray="20 20" />
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
	The goal is do decide if a given sample <code>x</code> is more likely to originate from one distribution (hypothesis 1) or from another one (hypthesis 2).
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