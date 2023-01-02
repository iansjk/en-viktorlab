<script lang="ts">
	import characterTable from '../viktorlab-fork/resources/gamedata/excel/character_table.json';
	import uniequipTable from '../viktorlab-fork/resources/gamedata/excel/uniequip_table.json';
	import battleEquipTable from '../viktorlab-fork/resources/gamedata/excel/battle_equip_table.json';
	import { calculateDps } from '../viktorlab-fork/resources/attributes.js';
	import dpsOptions from '../viktorlab-fork/resources/customdata/dps_options.json';

	const operators = Object.entries(characterTable)
		.filter(([id, op]) => id.startsWith('char_') && op.rarity > 1)
		.map(([id, op]) => ({ id, ...op }))
		.sort((a, b) => b.rarity - a.rarity);

	let operatorId: string | null = null;
	$: op = (operatorId ? characterTable[operatorId as keyof typeof characterTable] : {}) as Partial<
		typeof characterTable['char_002_amiya']
	>;
	$: rarity = (op?.rarity ?? 0) + 1;
	$: maxElite = (() => {
		if (rarity >= 4) {
			return 2;
		} else if (rarity > 2) {
			return 1;
		}
		return 0;
	})();

	$: elite = maxElite;

	$: maxLevel = (() => {
		if (elite === 2) {
			switch (rarity) {
				case 6:
					return 90;
				case 5:
					return 80;
				case 4:
					return 70;
			}
		} else if (elite === 1) {
			switch (rarity) {
				case 6:
					return 80;
				case 5:
					return 70;
				case 4:
					return 60;
				case 3:
					return 55;
			}
		}
		// elite 0
		switch (rarity) {
			case 6:
			case 5:
				return 50;
			case 4:
				return 45;
			case 3:
				return 40;
		}
		return 30;
	})();

	$: level = maxLevel;

	$: skills = op?.skills?.map((sk) => sk.skillId) ?? [];
	$: moduleIds = (
		uniequipTable.charEquip[operatorId as keyof typeof uniequipTable.charEquip] ?? []
	).filter((moduleId) => !moduleId.startsWith('uniequip_001'));

	let potential: number | null = 1;
	let trust: number | null = 100;
	$: skillId = skills.at(-1);
	$: skillLevel = rarity >= 4 ? 10 : 7;
	$: moduleId = moduleIds[0];
	$: maxModuleLevel =
		battleEquipTable[moduleId as keyof typeof battleEquipTable]?.phases?.length ?? 0;
	$: moduleLevel = maxModuleLevel;

	$: operatorDpsOptionsTagList = (
		operatorId ? dpsOptions.char[operatorId as keyof typeof dpsOptions.char] : []
	) as Array<keyof typeof dpsOptions.tags>;
	let operatorDpsOptions: Partial<Record<keyof typeof dpsOptions.tags, boolean>> = {
		buff: true
	};

	let enemyConfig: Exclude<Parameters<typeof calculateDps>[1], undefined> = {
		def: 0,
		magicResistance: 0,
		count: 1
	};

	let raidBuff: Exclude<Parameters<typeof calculateDps>[2], undefined> = {
		atk: 0,
		atkpct: 0,
		ats: 0,
		base_atk: 0,
		cdr: 0,
		damage_scale: 0
	};

	$: calculateDpsResult = operatorId
		? calculateDps(
				{
					charId: operatorId,
					phase: elite,
					level,
					potentialRank: (potential ?? 1) - 1,
					favor: trust ?? 100,
					skillId: skillId ?? '',
					skillLevel: skillLevel - 1,
					equipId: moduleId,
					equipLevel: moduleLevel,
					options: operatorDpsOptions
				},
				enemyConfig,
				raidBuff
		  )
		: null;
</script>

<div id="calc">
	<div>
		<section>
			<h2>Operator config</h2>
			<label>
				Operator
				<select bind:value={operatorId}>
					{#each operators as op (op.id)}
						<option value={op.id}>{op.name}</option>
					{/each}
				</select>
			</label>

			{#if operatorId}
				<label>
					Elite
					<input bind:value={elite} type="number" min="0" max={maxElite} />
				</label>
				<label>
					Level
					<input bind:value={level} type="number" min="1" max={maxLevel} />
				</label>
				<label>
					Potential
					<input bind:value={potential} type="number" min="0" max="6" />
				</label>
				<label>
					Trust
					<input bind:value={trust} type="number" min="0" max="200" />
				</label>

				{#if skills.length > 0}
					<label>
						Skill
						<select bind:value={skillId}>
							{#each skills as skillId}
								<option value={skillId}>{skillId}</option>
							{/each}
						</select>
					</label>

					<label>
						Skill Level
						<select bind:value={skillLevel}>
							{#each [...Array(rarity === 3 ? 7 : 10).keys()].map((i) => i + 1) as skLevel (skLevel)}
								<option value={skLevel}>{skLevel > 7 ? `M${skLevel - 7}` : skLevel}</option>
							{/each}
						</select>
					</label>
				{/if}

				{#if moduleIds.length > 0}
					<label>
						Module
						<select bind:value={moduleId}>
							{#each moduleIds as moduleId}
								<option value={moduleId}>{moduleId}</option>
							{/each}
						</select>
					</label>

					<label>
						Module Level
						<input type="number" bind:value={moduleLevel} min="1" max={maxModuleLevel} />
					</label>
				{/if}

				{#if operatorDpsOptionsTagList.length > 0}
					<h3>Other options</h3>
					{#each operatorDpsOptionsTagList as tag (tag)}
						<label>
							<input type="checkbox" bind:checked={operatorDpsOptions[tag]} />
							{tag}
						</label>
					{/each}
				{/if}
			{/if}
		</section>

		<section>
			<h2>Enemy config</h2>
			<label>
				Defense
				<input type="number" bind:value={enemyConfig.def} min="0" />
			</label>

			<label>
				Arts Resistance
				<input type="number" bind:value={enemyConfig.magicResistance} min="0" max="100" />
			</label>

			<label>
				Enemy count
				<input type="number" bind:value={enemyConfig.count} min="0" max="100" />
			</label>
		</section>

		<section>
			<h2>Buffs</h2>
			<label>
				Buffs enabled?
				<input type="checkbox" bind:checked={operatorDpsOptions.buff} />
			</label>

			<label>
				&#177;Attack (flat increase/decrease)
				<input type="number" bind:value={raidBuff.atk} />
			</label>

			<label>
				&#177;Attack % (percent increase/decrease)
				<input type="number" bind:value={raidBuff.atkpct} />
			</label>

			<label>
				&#177;ASPD
				<input type="number" bind:value={raidBuff.ats} />
			</label>

			<label>
				Base Attack &#177;% (percentage increase/decrease, e.g. Contingency Contract)
				<input type="number" bind:value={raidBuff.base_atk} />
			</label>

			<label>
				Skill Recovery &#177;%
				<input type="number" bind:value={raidBuff.cdr} />
			</label>

			<label>
				Attack scale &#177;%
				<input type="number" bind:value={raidBuff.damage_scale} />
			</label>
		</section>
	</div>

	{#if calculateDpsResult}
		<section>
			<h2>Results</h2>

			<dl>
				<div>
					<dt>Average DPS</dt>
					<dd>{calculateDpsResult.globalDps}</dd>
				</div>

				<div>
					<dt>Skill DPS</dt>
					<dd>{calculateDpsResult.skill.dps}</dd>
				</div>

				<div>
					<dt>Skill Attack Damage</dt>
					<dd>{calculateDpsResult.skill.atk}</dd>
				</div>

				<div>
					<dt>Basic Attack DPS</dt>
					<dd>{calculateDpsResult.normal.dps}</dd>
				</div>

				<div>
					<dt>Basic Attack Damage</dt>
					<dd>{calculateDpsResult.normal.atk}</dd>
				</div>
			</dl>

			<pre>
				{JSON.stringify(calculateDpsResult, null, 2)}
			</pre>
		</section>
	{/if}
</div>

<style>
	#calc {
		display: grid;
		grid-template-columns: 1fr 1fr;
		column-gap: 2rem;
	}

	#calc section {
		display: flex;
		flex-direction: column;
	}

	#calc label {
		display: grid;
		grid-template-columns: 2fr 1fr;
	}

	dl > div {
		display: grid;
		grid-template-columns: 1fr 1fr;
	}
</style>
