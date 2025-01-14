<script lang="ts">
    import { Buff, BuffDetails, type Skill, type StatusEffect } from "$lib/types";
    import { HexToRgba } from "$lib/utils/colors";
    import BuffTooltipDetail from "./shared/BuffTooltipDetail.svelte";
    import { tweened } from "svelte/motion";
    import { cubicOut } from "svelte/easing";
    import { skillIcon } from "$lib/utils/settings";
    import { getSkillIcon } from "$lib/utils/strings";
    import { tooltip } from "$lib/utils/tooltip";
    import { round } from "$lib/utils/numbers";

    export let skill: Skill;
    export let color: string;
    export let damagePercentage: number;
    export let groupedSynergies: Map<string, Map<number, StatusEffect>>;

    let synergyPercentageDetails: Array<BuffDetails>;

    const tweenedValue = tweened(0, {
        duration: 400,
        easing: cubicOut
    });

    $: {
        tweenedValue.set(damagePercentage);
        if (groupedSynergies.size > 0) {
            synergyPercentageDetails = [];
            groupedSynergies.forEach((synergies, key) => {
                let synergyDamage = 0;
                let buff = new BuffDetails();
                buff.id = key;
                synergies.forEach((syn, id) => {
                    if (skill.buffedBy[id]) {
                        buff.buffs.push(
                            new Buff(
                                syn.source.icon,
                                round((skill.buffedBy[id] / skill.totalDamage) * 100),
                                syn.source.skill?.icon
                            )
                        );
                        synergyDamage += skill.buffedBy[id];
                    } else if (skill.debuffedBy[id]) {
                        buff.buffs.push(
                            new Buff(
                                syn.source.icon,
                                round((skill.debuffedBy[id] / skill.totalDamage) * 100),
                                syn.source.skill?.icon
                            )
                        );
                        synergyDamage += skill.debuffedBy[id];
                    }
                });

                if (synergyDamage > 0) {
                    buff.percentage = round((synergyDamage / skill.totalDamage) * 100);
                }
                synergyPercentageDetails.push(buff);
            });
        }
    }
</script>

<tr class="h-7 px-2 py-1 text-3xs">
    <td class="pl-1">
        <img
            class="h-5 w-5"
            src={$skillIcon.path + getSkillIcon(skill.icon)}
            alt={skill.name}
            use:tooltip={{ content: skill.name }} />
    </td>
    <td colspan="2">
        <div class="truncate">
            {skill.name}
        </div>
    </td>
    {#if groupedSynergies.size > 0}
        {#each synergyPercentageDetails as synergy (synergy.id)}
            <td class="px-1 text-center">
                {#if synergy.percentage}
                    <BuffTooltipDetail {synergy} />
                {/if}
            </td>
        {/each}
    {/if}
    <div
        class="absolute left-0 -z-10 h-7 px-2 py-1"
        style="background-color: {HexToRgba(color, 0.6)}; width: {$tweenedValue}%" />
</tr>
