<script lang="ts">
	type FinancialYear = '2024-25' | '2025-26';

	interface TaxSlab {
		min: number;
		max: number;
		rate: number;
	}

	const TAX_SLABS: Record<FinancialYear, TaxSlab[]> = {
		'2024-25': [
			{ min: 0, max: 300000, rate: 0 },
			{ min: 300000, max: 700000, rate: 0.05 },
			{ min: 700000, max: 1000000, rate: 0.1 },
			{ min: 1000000, max: 1200000, rate: 0.15 },
			{ min: 1200000, max: 1500000, rate: 0.2 },
			{ min: 1500000, max: Infinity, rate: 0.3 }
		],
		'2025-26': [
			{ min: 0, max: 400000, rate: 0 },
			{ min: 400000, max: 800000, rate: 0.05 },
			{ min: 800000, max: 1200000, rate: 0.1 },
			{ min: 1200000, max: 1600000, rate: 0.15 },
			{ min: 1600000, max: 2000000, rate: 0.2 },
			{ min: 2000000, max: 2400000, rate: 0.25 },
			{ min: 2400000, max: Infinity, rate: 0.3 }
		]
	};

	function formatTaxSlabs(slabs: TaxSlab[]) {
		return slabs.map((slab, index) => {
			const min = slab.min / 100000;
			const max = slab.max / 100000;

			return {
				range: slab.max !== Infinity ? `₹${min === 0 ? '0' : min}L - ₹${max}L` : `₹${min}L +`,
				rate: `${slab.rate * 100}%`
			};
		});
	}

	let income: number;
	let fy: FinancialYear = '2025-26';
	let tax: number = 0;
	let cess: number = 0;
	let surcharge: number = 0;
	let breakdown: Array<{ slab: string; tax: number }> = [];

	$: calculateTax(income || 0), [fy, income];

	function calculateTax(amount: number) {
		const slabs = TAX_SLABS[fy];
		let totalTax = 0;
		const newBreakdown: Array<{ slab: string; tax: number }> = [];

		let remainingAmount = amount;

		for (const slab of slabs) {
			// if amount less than 7l or 12l then no tax based on fy
			if (amount <= (fy === '2024-25' ? 700000 : 1200000)) {
				totalTax = 0;
				break;
			}

			if (remainingAmount <= 0) break;

			const taxableInSlab = Math.min(remainingAmount, slab.max - slab.min);
			const taxInSlab = taxableInSlab * slab.rate;

			if (taxInSlab > 0) {
				newBreakdown.push({
					slab:
						(slab.max === Infinity
							? `₹${slab.min / 100000}L + `
							: `₹${slab.min / 100000}L - ₹${slab.max / 100000}L `) + `(${slab.rate * 100}%)`,

					tax: taxInSlab
				});
			}

			totalTax += taxInSlab;
			remainingAmount -= taxableInSlab;
		}

		// Calculate surcharge
		let surchargeRate = 0;
		if (amount > 20000000) surchargeRate = 0.25;
		else if (amount > 10000000) surchargeRate = 0.15;
		else if (amount > 5000000) surchargeRate = 0.1;

		surcharge = totalTax * surchargeRate;
		cess = (totalTax + surcharge) * 0.04;

		tax = totalTax;
		breakdown = newBreakdown;
	}

	function formatCurrency(amount: number) {
		return new Intl.NumberFormat('en-IN', {
			style: 'currency',
			currency: 'INR',
			maximumFractionDigits: 0
		}).format(amount);
	}

	$: totalTaxPayable = tax + surcharge + cess;
	$: effectiveTaxRate = income ? ((totalTaxPayable / income) * 100).toFixed(2) : '0';
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-50 p-6 border-t border-gray-200">
	<div class="max-w-3xl mx-auto">
		<div class="bg-white rounded-2xl shadow-xl p-8">
			<div class="flex items-center gap-3 mb-8">
				<iconify-icon icon="lucide:calculator" class="text-4xl text-indigo-600"></iconify-icon>
				<h1 class="text-3xl font-bold text-gray-800">Indian Income Tax Calculator</h1>
			</div>

			<div class="mb-8">
				<div class="flex gap-4 mb-4">
					<div class="flex-1">
						<label for="income" class="block text-sm font-medium text-gray-700 mb-2">
							Annual Income (₹)
						</label>
						<input
							type="number"
							id="income"
							bind:value={income}
							class="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500"
							placeholder="Enter your annual income"
						/>
					</div>
					<div>
						<label for="fy" class="block text-sm font-medium text-gray-700 mb-2">
							Financial Year
						</label>
						<select
							id="fy"
							bind:value={fy}
							class="w-40 px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 bg-white"
						>
							<option value="2024-25">FY 2024-25</option>
							<option value="2025-26">FY 2025-26</option>
						</select>
					</div>
				</div>
			</div>

			<div class="bg-indigo-50 rounded-xl p-6 mb-8">
				<h2 class="text-xl font-semibold text-gray-800 mb-4">Tax Summary</h2>
				<div class="space-y-2">
					<div class="flex justify-between items-center">
						<span class="text-gray-600">Base Tax:</span>
						<span class="font-semibold text-gray-800">{formatCurrency(tax)}</span>
					</div>
					<div class="flex justify-between items-center">
						<span class="text-gray-600">Health & Education Cess (4%):</span>
						<span class="font-semibold text-gray-800">{formatCurrency(cess)}</span>
					</div>
					{#if surcharge > 0}
						<div class="flex justify-between items-center">
							<span class="text-gray-600">Surcharge:</span>
							<span class="font-semibold text-gray-800">{formatCurrency(surcharge)}</span>
						</div>
					{/if}
					<div class="h-px bg-gray-200 my-2"></div>
					<div class="flex justify-between items-center">
						<span class="text-gray-700 font-medium">Total Tax Payable:</span>
						<span class="text-2xl font-bold text-indigo-600">{formatCurrency(totalTaxPayable)}</span
						>
					</div>
				</div>
				<div class="text-sm text-gray-600 mt-4">
					Effective Tax Rate: {effectiveTaxRate}%
				</div>
			</div>

			{#if breakdown.length > 0}
				<div>
					<h3 class="text-lg font-semibold text-gray-800 mb-4">Tax Breakdown</h3>
					<div class="space-y-3">
						{#each breakdown as item}
							<div class="flex justify-between items-center bg-gray-50 p-3 rounded-lg">
								<span class="text-gray-700">{item.slab}</span>
								<span class="font-medium text-gray-900">{formatCurrency(item.tax)}</span>
							</div>
						{/each}
					</div>
				</div>
			{/if}
		</div>

		<div class="mt-8 bg-white rounded-2xl shadow-xl p-8">
			<h2 class="text-xl font-semibold text-gray-800 mb-4">Tax Slabs (FY {fy})</h2>
			<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
				{#each formatTaxSlabs(TAX_SLABS[fy]) as slab}
					<div class="flex justify-between bg-gray-50 p-4 rounded-lg">
						<span class="text-gray-700">{slab.range}</span>
						<span class="font-medium text-gray-900">{slab.rate}</span>
					</div>
				{/each}
			</div>
			<div class="mt-6">
				<h3 class="text-lg font-semibold text-gray-800 mb-4">Surcharge Rates</h3>
				<div class="grid grid-cols-1 md:grid-cols-2 gap-4">
					{#each [{ range: '₹50L - ₹1Cr', rate: '10%' }, { range: '₹1Cr - ₹2Cr', rate: '15%' }, { range: 'Above ₹2Cr', rate: '25%' }] as slab}
						<div class="flex justify-between bg-gray-50 p-4 rounded-lg">
							<span class="text-gray-700">{slab.range}</span>
							<span class="font-medium text-gray-900">{slab.rate}</span>
						</div>
					{/each}
				</div>
			</div>
			<div class="mt-4 text-sm text-gray-600">
				<p>
					Note: 4% Health & Education Cess is applicable on the total tax amount (including
					surcharge).
				</p>
				<p class="mt-2 font-medium text-indigo-600">
					Special Rebate: No tax payable for total income up to
					{fy === '2024-25' ? '₹7 Lakhs' : '₹12 Lakhs'}
				</p>
			</div>
		</div>
	</div>
</div>
