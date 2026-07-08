---
name: progressive-diagramming
description: Use when the user asks to draw, make, generate, optimize, or fix diagrams/flowcharts/architecture diagrams/Draw.io/Mermaid/PlantUML/Graphviz/Excalidraw. Progressively structure the diagram before rendering so connector lines stay clean.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  source: user-global
  tags: [diagramming, flowcharts, mermaid, drawio, architecture, visualization]
---

# Progressive Diagramming

## Overview

Use this skill whenever the user mentions drawing diagrams, flowcharts, architecture diagrams, sequence diagrams, Draw.io, Mermaid, PlantUML, Graphviz, Excalidraw, or complains that AI-generated diagram connectors are messy.

The core rule is: **do not jump directly to a free-canvas diagram**. First turn the user's idea into a clean structure, then pick the simplest diagram language/tool that preserves layout.

## Progressive Trigger

Trigger lightly when the user says words like:

- 画图、流程图、架构图、时序图、状态图、泳道图、拓扑图
- draw.io、Next AI Draw.io、Mermaid、PlantUML、Graphviz、Excalidraw
- 线条乱、连接线乱、布局乱、交叉线、图不清楚、图太复杂

If the user only asks for tool recommendations, answer with recommendations. If the user asks you to create or improve a diagram, apply the workflow below.

## Workflow

1. **Classify the diagram type**
   - Business/process flow → Mermaid `flowchart TD`
   - System calls or interactions → Mermaid/PlantUML sequence diagram
   - State transitions → Mermaid state diagram or PlantUML state diagram
   - Architecture overview for presentation → Excalidraw or SVG/HTML artifact
   - Dense dependency graph → Graphviz DOT
   - Editable final business artifact → Draw.io after structure is stable

   Completion: diagram type and output format are explicit.

2. **Extract structure before drawing**
   - Identify actors/systems, stages, decisions, success paths, failure paths, retries, and terminal states.
   - Output a short layer list when the flow is non-trivial:
     - L1 start/trigger
     - L2 input/validation
     - L3 processing
     - L4 decisions
     - L5 success/failure handling
     - L6 end/output

   Completion: every node belongs to one layer or one lane.

3. **Apply clean-layout rules**
   - Prefer top-down (`TD`) for process flows.
   - Keep the main path centered and vertical.
   - Put failures, exceptions, retries, and compensation on the right side.
   - Put optional side notes on the left or in a separate note block.
   - Each layer should have at most 3 nodes unless the user explicitly wants detail.
   - Avoid cross-layer jump lines; connect adjacent layers whenever possible.
   - Use short node labels, usually ≤12 Chinese characters or ≤6 English words.

   Completion: no obvious connector crossings or long diagonal edges remain.

4. **Choose output format deliberately**
   - Default to Mermaid for clean, maintainable flowcharts.
   - Use PlantUML when sequence/state detail matters more than visual styling.
   - Use Graphviz when automatic layout matters more than easy editing.
   - Use Excalidraw when the user wants a hand-drawn presentation artifact.
   - Use Draw.io only after the structure is stable, or when the user specifically needs editable Draw.io.

   Completion: the format matches the user's editing/rendering needs.

5. **For Draw.io requests, constrain generation**
   If the user insists on Draw.io/Next AI Draw.io, include these constraints in the prompt or generated structure:
   - Use top-to-bottom layout, not free scattered placement.
   - Main flow stays on the center vertical line.
   - Exception/failure branches stay on the right.
   - Retry/back edges are dashed and placed on the right.
   - Avoid diagonal connectors; use orthogonal/elbow connectors.
   - Leave at least 80px between nodes.
   - Connect lines to the center of node edges, not corners.
   - First output node hierarchy, then generate the diagram.

   Completion: Draw.io receives a constrained structure rather than a vague prompt.

6. **Review before finalizing**
   - Check whether the graph has too many nodes for one diagram.
   - If >15 nodes, >3 decision nodes, or many retries/exceptions, recommend splitting into main flow + exception flow + state/sequence diagram.
   - If the output is Mermaid/PlantUML/DOT, provide only the code block plus a short usage note unless the user asks for explanation.

   Completion: final output is readable, not just technically valid.

## Reusable Prompt

Use or adapt this prompt when the user wants better AI-generated diagrams:

```text
请先不要画图，先把流程拆成层级结构。然后用 Mermaid flowchart TD 生成流程图。要求主流程从上到下，异常分支在右侧，判断节点不超过3个，每层最多3个节点，禁止跨层跳线，禁止交叉线，节点文案短句化。最后检查是否存在交叉连接，如有则重新布局。
```

For Draw.io/Next AI Draw.io:

```text
请生成 draw.io 流程图，但必须遵守：从上到下布局；每层最多3个节点；主流程走中间竖线；异常/失败分支统一放右侧；回退/重试分支用虚线并放右侧；禁止连接线交叉；禁止斜线连接，尽量使用正交折线；节点间距至少80px；所有线条连接到节点中心边缘，不连接角；先输出节点层级结构，再生成图。
```

## Common Pitfalls

1. **Starting with Draw.io too early.** Free-canvas generation makes messy connectors likely. Stabilize structure in Mermaid/PlantUML first.
2. **Too many details in one picture.** Split large diagrams instead of forcing everything into one canvas.
3. **Long node labels.** Long labels force wide nodes and cause routing problems. Shorten labels and put details below the diagram.
4. **Mixed diagram purposes.** Do not combine process flow, sequence calls, architecture topology, and data model in one diagram unless the user explicitly wants a map.
5. **Ignoring exception paths.** Place exception paths consistently on the right so the main path stays readable.

## Verification Checklist

- [ ] Diagram type and output format are explicit
- [ ] Nodes are grouped into layers, lanes, or states before rendering
- [ ] Main path is visually dominant and mostly vertical/top-down
- [ ] Exception/retry paths are consistently placed
- [ ] Labels are short enough to avoid layout sprawl
- [ ] No obvious connector crossings remain
- [ ] Large diagrams are split or a split is recommended
