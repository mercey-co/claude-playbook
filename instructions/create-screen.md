---
description: Design System Screen Creation Rules
globs:
alwaysApply: false
version: 1.0
encoding: UTF-8
---

# Screen Creation Rules (React Native)

<ai_meta>
  <rules>Assemble React Native app screens strictly from DS links, preserve exact naming (components & variants), enforce DS token usage, block any invention (colors, fonts, spacing, icons, layouts)</rules>
  <format>UTF-8, LF, 2-space indent, no header indent</format>
</ai_meta>

## Overview

Create a **React Native app screen** strictly using the provided Figma Design System (via MCP).  
Do not invent, rename, or approximate. Use **exact DS naming** for components/variants, layouts, and tokens (colors, typography, spacing, radii).  
If something is missing from the DS, stop and output a `TODO: missing_*`.

<agent_detection>
  <check_once>
    AT START OF PROCESS:
    SET has_figma_agent = (Claude Code AND figma-ds-implementer agent exists)
    USE this flag throughout execution
  </check_once>
</agent_detection>

<process_flow>

<step number="1" name="gather_user_input">

### Step 1: Gather User Input

<step_metadata>
  <required_inputs>
    - screen_name: string (exact DS page/screen name)
    - route_id: string (app navigation route)
    - ds_links: array[url] (minimum: 1; must include node-id or fileKey)
    - purpose: string (screen’s role in the flow)
    - user_flows: array[string] (entry → actions → exit)
  </required_inputs>
  <validation>blocking</validation>
</step_metadata>

<instructions>
  Collect name, route, DS references, and flow purpose.
  ERROR if any are missing → show error_template.
</instructions>

<error_template>
  Please provide the following missing information:
  1. Screen name (exact, per DS)
  2. App route identifier
  3. DS link(s) with nodeId or fileKey
  4. Purpose of the screen
  5. User flow entry/actions/exit
</error_template>

</step>

<step number="2" name="validate_ds_references">

### Step 2: Validate DS References

<instructions>
  IF has_figma_agent:
    CALL per ds_link:
      - get_variable_defs
      - get_code_connect_map
    VALIDATE: 
      - Screen layout exists in DS
      - All referenced DS components are resolvable
      - Variants exist (exact names only)
      - Tokens exist for layout grid, color, typography, spacing, radius
  ELSE:
    ERROR: Cannot proceed without figma-ds-implementer agent
</instructions>

</step>

<step number="3" name="assemble_react_native_screen">

### Step 3: Assemble React Native Screen

<file_structure>
  src/
  └── screens/
      └── [SCREEN_NAME]/
          ├── [SCREEN_NAME].tsx         # RN screen (strict DS assembly)
          ├── index.ts                  # re-export
          ├── layout.map.ts             # DS layout + grid token bindings
          ├── components.map.ts         # DS component instances on this screen
          ├── README.md                 # usage, DS references, flows
          └── [SCREEN_NAME].test.tsx    # tests (states & flows)
</file_structure>

<file_template name="[SCREEN_NAME].tsx">
  import React from 'react';
  import { View } from 'react-native';
  import { layoutFromTokens } from './layout.map';
  import { Components } from './components.map';

  export const [SCREEN_NAME]: React.FC = () => {
    const s = layoutFromTokens();

    return (
      <View style={s.container}>
        {/* DS component instances only */}
        <Components />
      </View>
    );
  };

  export default [SCREEN_NAME];
</file_template>

<file_template name="layout.map.ts">
  // Strict DS layout grid bindings (spacing, columns, gutters, breakpoints)
  import { StyleSheet } from 'react-native';

  export function layoutFromTokens() {
    return StyleSheet.create({
      container: {
        flex: 1,
        // Use DS grid + spacing tokens only
      },
    });
  }
</file_template>

<file_template name="components.map.ts">
  // Assembly of DS components for this screen
  // No custom components, no invention
  export function Components() {
    return (
      <>
        {/* Example: <Button state="Primary" size="Md" __variantExact="Button/Primary/Md" /> */}
        {/* Filled strictly from DS references */}
      </>
    );
  }
</file_template>

<file_template name="README.md">
  # [SCREEN_NAME] (React Native)

  - **DS exact name:** [SCREEN_NAME]
  - **Route:** [ROUTE_ID]
  - **Purpose:** [PURPOSE]
  - **DS References:**
    - [DS_LINK_1]
    - [DS_LINK_2]
  - **User Flows:** [ENTRY → ACTIONS → EXIT]
  - **Tokens:** (no inline values)
    - layout/grid: [DS_TOKEN]
    - spacing: [DS_TOKEN]
    - typography: [DS_TOKEN]
  - **Code Connect:**
    - [CODE_CONNECT_MAP]

  > Strict policy: No invention. No renaming. DS tokens only.
</file_template>

</step>

</process_flow>

## Execution Summary

<final_checklist>
  <verify>
    - [ ] Screen name, route, DS links collected
    - [ ] DS references validated via MCP
    - [ ] All components resolved from DS
    - [ ] Tokens mapped (grid, spacing, colors, typography)
    - [ ] RN files created and tests added
    - [ ] TODOs flagged where DS incomplete
  </verify>
</final_checklist>

<execution_order>
  1. Gather user inputs
  2. Validate DS links (MCP)
  3. Generate React Native files
  4. Populate tokens/layout/components from DS
  5. Stop with TODOs if DS incomplete
</execution_order>