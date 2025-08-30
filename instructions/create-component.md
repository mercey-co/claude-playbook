---
description: Design System Component Creation Rules
globs:
alwaysApply: false
version: 1.0
encoding: UTF-8
---

# Component Creation Rules (React Native)

<ai_meta>
  <rules>Assemble React Native components strictly from DS links, preserve exact naming (components & variants), enforce DS token usage, block any invention (colors, fonts, spacing, icons)</rules>
  <format>UTF-8, LF, 2-space indent, no header indent</format>
</ai_meta>

## Overview

Create a **React Native** component strictly using the provided Figma Design System (via MCP).  
Do not invent, rename, or approximate. Enforce **exact DS naming** for components/variants and **tokens only** (colors, typography, spacing, radii).  
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
    - component_name: string (exact DS name)
    - intended_usage: string (screen/flow)
    - ds_links: array[url] (minimum: 1; must include node-id or fileKey)
    - variant_options: array[string] (optional; must match DS variants exactly)
  </required_inputs>
  <validation>blocking</validation>
</step_metadata>

<data_sources>
  <primary>user_direct_input</primary>
</data_sources>

<error_template>
  Please provide the following missing information:
  1. Component name (exact, per DS)
  2. Intended usage (what screen/flow)
  3. DS link(s) with nodeId or fileKey
  4. Variant options (if applicable, exact DS names)
</error_template>

<instructions>
  ACTION: Collect required inputs from the user
  VALIDATION: Ensure at least name, usage, and DS link(s) are provided
  ERROR: Use error_template if inputs are missing
</instructions>

</step>

<step number="2" name="validate_ds_references">

### Step 2: Validate DS References

<step_metadata>
  <requires>
    - ds_links
  </requires>
</step_metadata>

<instructions>
  IF has_figma_agent:
    USE: @agent:figma-ds-implementer
    CALL per ds_link:
      - get_variable_defs
      - get_code_connect_map
    VALIDATE: 
      - Component exists in DS (node resolvable)
      - Requested variants exist (exact names)
      - Tokens exist for color/typography/spacing/radius
      - (If mapped) codeConnectName/src present and must be preserved
  ELSE:
    ERROR: Cannot proceed without figma-ds-implementer agent
</instructions>

<error_template>
  One or more DS references are invalid or missing. Please verify the links provided.
</error_template>

</step>

<step number="3" name="assemble_react_native_component">

### Step 3: Assemble React Native Component

<step_metadata>
  <creates>
    - directory: src/components/[COMPONENT_NAME]/
    - files: 6
  </creates>
</step_metadata>

<file_structure>
  src/
  └── components/
      └── [COMPONENT_NAME]/
          ├── [COMPONENT_NAME].tsx          # RN component (strict DS)
          ├── index.ts                      # re-export
          ├── tokens.map.ts                 # DS token bindings (no inline values)
          ├── variants.ts                   # exact DS variants as TypeScript unions
          ├── README.md                     # usage, props, DS references
          └── [COMPONENT_NAME].test.tsx     # tests (props/variants/tokens)
</file_structure>

<file_template name="[COMPONENT_NAME].tsx">
  import React from 'react';
  import { Pressable, Text, View, ViewStyle, TextStyle } from 'react-native';
  import { stylesFromTokens } from './tokens.map';
  import { VariantProps, assertVariant } from './variants';

  export type [COMPONENT_NAME]Props = VariantProps & {
    label?: string;            // only if DS defines label/text slot
    iconLeft?: React.ReactNode;  // only if DS includes icon-left slot
    iconRight?: React.ReactNode; // only if DS includes icon-right slot
    style?: ViewStyle;         // optional container override (layout only)
    textStyle?: TextStyle;     // optional text layout override (no color/font)
    onPress?: () => void;      // if DS indicates interactivity
    disabled?: boolean;        // if DS defines disabled state
    // No arbitrary props that introduce non-DS styles
  };

  export const [COMPONENT_NAME]: React.FC<[COMPONENT_NAME]Props> = (props) => {
    assertVariant(props); // throws if variant not in DS
    const s = stylesFromTokens(props); // derives RN styles from DS tokens only

    const Content = (
      <View style={s.container}>
        {props.iconLeft ? <View style={s.iconLeft}>{props.iconLeft}</View> : null}
        {props.label ? <Text style={s.label}>{props.label}</Text> : null}
        {props.iconRight ? <View style={s.iconRight}>{props.iconRight}</View> : null}
      </View>
    );

    if (typeof props.onPress === 'function') {
      return (
        <Pressable
          accessibilityRole="button"
          accessibilityState={{ disabled: !!props.disabled }}
          disabled={props.disabled}
          style={({ pressed }) => [s.pressable, pressed ? s.pressed : null, props.style]}
          onPress={props.onPress}
        >
          {Content}
        </Pressable>
      );
    }

    return <View style={[s.block, props.style]}>{Content}</View>;
  };

  export default [COMPONENT_NAME];
</file_template>

<file_template name="tokens.map.ts">
  // DS token bindings — no inline colors/fonts/sizes allowed.
  // Pull all values from DS variables resolved via MCP.
  import { StyleSheet } from 'react-native';
  import { ExactVariant, VariantProps } from './variants';

  // Example shape — replace with DS-resolved token keys.
  type TokenRefs = {
    colorBg: string;
    colorText: string;
    spacingX: number;
    spacingY: number;
    radius: number;
    fontFamily: string;
    fontSize: number;
    fontWeight: string | number;
    lineHeight: number;
    iconGap: number;
    pressedOverlay?: string; // optional token if DS defines pressed/hover
  };

  const TOKENS: Record<ExactVariant, TokenRefs> = {
    // Fill from MCP get_variable_defs — keys must match DS token names exactly.
    // e.g. 'Button/Primary/Md': { colorBg: var('color/button/primary/bg'), ... }
  } as const;

  export function stylesFromTokens(p: VariantProps) {
    const t = TOKENS[p.__variantExact]; // exact DS variant key
    if (!t) throw new Error(`Missing DS tokens for variant: ${p.__variantExact}`);

    return StyleSheet.create({
      block: { /* layout only; never color/font here */ },
      pressable: {
        borderRadius: t.radius,
        backgroundColor: t.colorBg,
      },
      pressed: t.pressedOverlay ? { backgroundColor: t.pressedOverlay } : {},
      container: {
        flexDirection: 'row',
        alignItems: 'center',
        gap: t.iconGap,
        paddingHorizontal: t.spacingX,
        paddingVertical: t.spacingY,
      },
      label: {
        color: t.colorText,
        fontFamily: t.fontFamily,
        fontSize: t.fontSize,
        fontWeight: t.fontWeight as any,
        lineHeight: t.lineHeight,
      },
      iconLeft: { },
      iconRight: { },
    });
  }
</file_template>

<file_template name="variants.ts">
  // Exact DS variants only — names/types must match DS exactly.
  export type Size = 'Sm' | 'Md' | 'Lg';           // replace from DS
  export type State = 'Primary' | 'Secondary';     // replace from DS
  export type WithIcon = 'None' | 'Left' | 'Right' | 'Both'; // replace from DS if exists
  export type Disabled = boolean;                  // if DS has disabled state

  // Compose exact DS variant key for token lookup (e.g., "Button/Primary/Md")
  export type ExactVariant = string; // enforce as literal unions after DS validation

  export type VariantProps = {
    size: Size;
    state: State;
    withIcon?: WithIcon;
    disabled?: Disabled;
    __variantExact: ExactVariant; // generated after validation; must match DS exactly
  };

  export function assertVariant(p: VariantProps): asserts p is VariantProps {
    // Validate against DS variant matrices discovered via MCP; throw on mismatch.
    // No fallback. No approximations.
  }
</file_template>

<file_template name="index.ts">
  export { default } from './[COMPONENT_NAME]';
  export * from './[COMPONENT_NAME]';
</file_template>

<file_template name="README.md">
  # [COMPONENT_NAME] (React Native)

  - **DS exact name:** [COMPONENT_NAME]
  - **Usage:** [INTENDED_USAGE]
  - **DS References:**
    - [DS_LINK_1]
    - [DS_LINK_2]
  - **Variants:** (exact DS)
    - size: [LIST]
    - state: [LIST]
    - withIcon: [LIST]
  - **Tokens:** (no inline values)
    - color/background: [DS_TOKEN]
    - color/text: [DS_TOKEN]
    - spacing: [DS_TOKEN]
    - radius: [DS_TOKEN]
    - typography: [DS_TOKEN_FAMILY | SIZE | WEIGHT | LINE_HEIGHT]
  - **Code Connect:**
    - name: [CODE_CONNECT_NAME]
    - src:  [CODE_CONNECT_SRC]

  > Strict policy: No invention. No renaming. Tokens only.
</file_template>

<file_template name="[COMPONENT_NAME].test.tsx">
  import React from 'react';
  import { render } from '@testing-library/react-native';
  import { [COMPONENT_NAME] } from './[COMPONENT_NAME]';

  describe('[COMPONENT_NAME]', () => {
    it('renders with exact DS variant tokens', () => {
      const { getByText } = render(
        <[COMPONENT_NAME]
          label="Submit"
          size="Md"
          state="Primary"
          __variantExact="Button/Primary/Md"
        />
      );
      expect(getByText('Submit')).toBeTruthy();
    });

    it('throws on non-DS variant', () => {
      expect(() =>
        render(
          <[COMPONENT_NAME]
            label="X"
            // @ts-expect-error non-DS size
            size="Medium"
            state="Primary"
            __variantExact="Button/Primary/Medium"
          />
        )
      ).toThrow();
    });
  });
</file_template>

<instructions>
  ACTIONS:
    - CREATE directory and files exactly as above
    - FILL token and variant maps from MCP:
      - get_variable_defs --nodeId=<DS component or page>
      - get_code_connect_map --nodeId=<DS component>
    - PRESERVE exact DS names for component and variants
    - FORBID inline color/font/spacing/radius — use DS tokens only
    - IF any DS element is missing → write TODO in README.md and STOP

  BLOCKERS:
    - Missing DS component/variant/token → `TODO: missing_*` and halt
    - No icons outside DS; no ad-hoc images
    - No custom styles beyond layout container overrides
</instructions>

</step>

</process_flow>

## Execution Summary

<final_checklist>
  <verify>
    - [ ] Component name, usage, DS links collected
    - [ ] DS references validated via MCP
    - [ ] Variants match DS exactly (names & matrix)
    - [ ] Tokens mapped; no inline values exist
    - [ ] Code Connect mapping preserved (if present)
    - [ ] RN files created and tests added
    - [ ] TODOs flagged where DS incomplete
  </verify>
</final_checklist>

<execution_order>
  1. Gather user inputs
  2. Validate DS links (MCP)
  3. Generate React Native files
  4. Populate tokens/variants from DS
  5. Stop with TODOs if DS incomplete
</execution_order>