<script lang="ts">
	let operatorId: string | null = null;
	let elite: number | null = null;
	let level: number | null = null;
	let potential: number | null = null;
	let trust: number | null = null;
	let moduleId: string | null = null;
	let moduleLevel: number | null = null;
	let skillId: string | null = null;
	let skillLevel: number | null = null;
</script>

{#if operatorId}
	<label>
		Elite
		<input bind:value={elite} type="number" min="0" max="2" />
	</label>
{/if}
