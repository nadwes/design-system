# Implementation Plan: White-Label Design System

## Overview

Implement the design system as a TypeScript package with a three-level token hierarchy, WCAG 2.1 AA accessibility validation, Figma Variables sync pipeline, and a brand registry. The DYN Sports brand is the reference implementation. Tasks are ordered so each step builds on the previous and all code is wired together before the end.

## Tasks

- [ ] 1. Set up project structure, tooling, and core TypeScript types
  - Initialize a TypeScript package with `package.json`, `tsconfig.json`, and a test runner (Vitest)
  - Create the directory structure: `src/tokens/`, `src/brands/`, `src/validation/`, `src/figma/`, `src/export/`, `tests/unit/`, `tests/property/`
  - Define all core TypeScript interfaces: `BaseToken`, `SemanticToken`, `ComponentToken`, `TokenSet`, `ColorScheme`, `BrandRegistryEntry`, `BrandRegistry`, `ValidationReport`, `ColorCheckResult`, `TypographyCheckResult`
  - Export all types from `src/index.ts`
  - Install `fast-check` for property-based testing
  - _Requirements: 1.1, 1.2, 1.3, 2.1_

- [ ] 2. Implement the token schema and schema validation
  - [ ] 2.1 Define `TOKEN_SCHEMA` constant in `src/tokens/schema.ts` with all required semantic and base token IDs per the design document
    - Include all color role tokens, typography style tokens, spacing semantic tokens, spacing base scale, border-radius, border-width, shadow, and motion tokens
    - _Requirements: 1.5, 2.2, 3.2, 4.2, 5.1, 5.3, 6.1, 6.2, 6.3, 6.4_

  - [ ] 2.2 Implement `validateSchema(tokenSet: TokenSet): SchemaValidationResult` in `src/tokens/schema.ts`
    - Check that every required semantic token ID is present in `tokenSet.semanticTokens`
    - Check that every required base token ID is present in `tokenSet.baseTokens`
    - Return a result listing any missing token IDs
    - _Requirements: 2.2, 2.3_

  - [ ]* 2.3 Write property test for schema completeness (Property 4)
    - **Property 4: Token set schema completeness**
    - **Validates: Requirements 2.2, 2.3**
    - Generate arbitrary token sets that pass schema validation and assert all required IDs are present
    - Generate token sets with randomly removed required tokens and assert they fail validation

  - [ ] 2.4 Implement `validateReferenceIntegrity(tokenSet: TokenSet): ReferenceIntegrityResult`
    - For every `SemanticToken`, verify its `ref` resolves to an existing `BaseToken` id
    - For every `ComponentToken`, verify its `ref` resolves to an existing `SemanticToken` id
    - Return a result listing any broken references
    - _Requirements: 1.2, 1.3_

  - [ ]* 2.5 Write property tests for reference integrity (Properties 1 and 2)
    - **Property 1: Semantic token reference integrity**
    - **Property 2: Component token reference integrity**
    - **Validates: Requirements 1.2, 1.3**
    - Generate arbitrary valid token sets and assert all references resolve
    - Generate token sets with deliberately broken refs and assert they fail

- [ ] 3. Implement token resolution and propagation
  - [ ] 3.1 Implement `resolveSemanticToken(tokenSet: TokenSet, semanticId: string): BaseTokenValue` in `src/tokens/resolver.ts`
    - Follow the `SemanticToken.ref` chain to return the resolved `BaseToken` value
    - _Requirements: 1.2, 1.4_

  - [ ] 3.2 Implement `resolveComponentToken(tokenSet: TokenSet, componentId: string): BaseTokenValue`
    - Follow `ComponentToken.ref` → `SemanticToken.ref` → `BaseToken.value`
    - _Requirements: 1.3, 1.4_

  - [ ] 3.3 Implement `updateBaseToken(tokenSet: TokenSet, baseId: string, newValue: BaseTokenValue): TokenSet`
    - Return a new `TokenSet` with the updated base token value (immutable update)
    - All downstream resolutions must reflect the new value without additional changes
    - _Requirements: 1.4_

  - [ ]* 3.4 Write property test for base token update propagation (Property 3)
    - **Property 3: Base token update propagation**
    - **Validates: Requirements 1.4**
    - For any token set and any base token update, assert that `resolveSemanticToken` and `resolveComponentToken` return the new value for all tokens that transitively reference the updated base token

- [ ] 4. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 5. Implement the DYN Sports brand token set
  - [ ] 5.1 Create `src/brands/dyn-sports/base-tokens.ts` with the full DYN Sports color palette, typography scale, spacing scale, border-radius, border-width, shadow, and motion base tokens
    - Color palette: primary orange, navy, neutral, success/warning/error/info semantics
    - Typography: 1.25 modular ratio font-size scale in rem, font-weight scale, line-height scale, letter-spacing scale
    - Spacing: 13-step scale (0–24) with base unit 0.25rem
    - Motion: include `motion.duration.none` with value `0ms`
    - _Requirements: 3.1, 4.1, 4.6, 5.1, 5.2, 6.1, 6.2, 6.3, 6.4, 6.5_

  - [ ] 5.2 Create `src/brands/dyn-sports/semantic-tokens.ts` with all semantic token definitions referencing base tokens
    - All color role tokens (action, text, surface, border, state)
    - All typography style tokens (display, heading.xl–sm, body.lg–sm, label.lg–sm, caption, overline)
    - All spacing semantic tokens (component padding sm/md/lg, layout gap sm/md/lg, section)
    - _Requirements: 3.2, 4.2, 5.3_

  - [ ] 5.3 Create `src/brands/dyn-sports/color-schemes.ts` defining light and dark `ColorScheme` objects
    - Light scheme: maps each semantic color token to its light-mode resolved hex value
    - Dark scheme: maps each semantic color token to its dark-mode resolved hex value
    - _Requirements: 3.7_

  - [ ] 5.4 Assemble and export the complete `DYN_SPORTS_TOKEN_SET: TokenSet` from `src/brands/dyn-sports/index.ts`
    - Run `validateSchema` and `validateReferenceIntegrity` at module load time (throw on failure)
    - _Requirements: 2.1, 2.2_

  - [ ]* 5.5 Write unit tests for DYN Sports token set completeness
    - Assert all required schema token IDs are present
    - Assert `motion.duration.none` value is `"0ms"`
    - Assert font-size base tokens are expressed in rem
    - Assert spacing base tokens follow the 0.25rem base unit rule
    - _Requirements: 2.1, 4.6, 5.2, 6.5_

- [ ] 6. Implement the default token set and brand extension mechanism
  - [ ] 6.1 Create `src/brands/default/index.ts` with a default `TokenSet` that serves as the base for brand extension
    - _Requirements: 2.4_

  - [ ] 6.2 Implement `extendTokenSet(base: TokenSet, overrides: Partial<TokenSet>, brandId: string, version: string): TokenSet` in `src/brands/registry.ts`
    - Merge base and override token sets; override values take precedence
    - The resulting token set must pass schema and reference integrity validation
    - _Requirements: 2.4_

  - [ ] 6.3 Implement `diffTokenSets(a: TokenSet, b: TokenSet): TokenSetDiff` in `src/brands/registry.ts`
    - Return the symmetric difference: tokens in A but not B, and tokens in B but not A
    - _Requirements: 2.5, 9.5_

  - [ ]* 6.4 Write property tests for extension and diff (Properties 5 and 6)
    - **Property 5: Token set extension correctness**
    - **Property 6: Token set diff completeness**
    - **Validates: Requirements 2.4, 2.5, 9.5**
    - For extension: assert that for any token in the override set, the extended set uses the override value; for all other tokens, the base value is used
    - For diff: assert the result equals the symmetric difference of token id sets

- [ ] 7. Implement WCAG contrast ratio validation
  - [ ] 7.1 Implement `hexToRgb(hex: string): { r: number; g: number; b: number }` in `src/validation/contrast.ts`
    - Handle 3-digit and 6-digit hex strings, with or without `#` prefix
    - _Requirements: 3.3, 3.4, 3.5_

  - [ ] 7.2 Implement `relativeLuminance(hex: string): number` using the WCAG 2.1 formula
    - _Requirements: 3.3, 3.4, 3.5_

  - [ ] 7.3 Implement `contrastRatio(fg: string, bg: string): number`
    - _Requirements: 3.3, 3.4, 3.5_

  - [ ]* 7.4 Write unit tests for contrast calculation
    - Test known pairs: white on black (21:1), `#767676` on white (≈4.54:1), `#949494` on white (≈3.03:1)
    - Test edge cases: same color (1:1), near-black on black
    - _Requirements: 3.3, 3.4, 3.5_

  - [ ] 7.5 Implement `validateColorPairs(tokenSet: TokenSet, scheme: "light" | "dark"): ColorCheckResult[]` in `src/validation/contrast.ts`
    - Check all color pairs defined in the design document against their required ratios
    - Return a `ColorCheckResult` for each pair
    - _Requirements: 3.3, 3.4, 3.5, 3.6, 8.1_

  - [ ]* 7.6 Write property tests for contrast compliance (Properties 7 and 8)
    - **Property 7: Normal text contrast compliance**
    - **Property 8: Large text and UI component contrast compliance**
    - **Validates: Requirements 3.3, 3.4, 3.5, 8.1**
    - Generate token sets where all color pairs are known-compliant and assert all checks pass
    - Generate token sets with at least one known-failing pair and assert the report marks it as failed

- [ ] 8. Implement typography accessibility validation
  - [ ] 8.1 Implement `validateTypography(tokenSet: TokenSet): TypographyCheckResult[]` in `src/validation/typography.ts`
    - Check body text font size ≥ 16px (1rem)
    - Check line-height ≥ 1.5 × font-size for body text tokens
    - Check letter-spacing ≥ 0.12 × font-size for body text tokens
    - _Requirements: 4.3, 4.4, 4.5, 8.2_

  - [ ]* 8.2 Write property test for typography spacing compliance (Property 9)
    - **Property 9: Typography spacing compliance**
    - **Validates: Requirements 4.3, 4.4, 4.5, 8.2**
    - Generate compliant typography token sets and assert all checks pass
    - Generate token sets with one failing typography value and assert the check fails

- [ ] 9. Implement the full accessibility validation pipeline
  - [ ] 9.1 Implement `validateTokenSet(tokenSet: TokenSet): ValidationReport` in `src/validation/validator.ts`
    - Run `validateColorPairs` for both light and dark schemes
    - Run `validateTypography`
    - Set `compliant: true` only if all checks pass
    - _Requirements: 8.1, 8.2, 8.3, 8.5_

  - [ ] 9.2 Implement `canPublish(report: ValidationReport): boolean`
    - Returns `true` only if `report.compliant === true`
    - _Requirements: 8.4_

  - [ ]* 9.3 Write unit tests for the validation pipeline
    - Test that a known-compliant token set produces `compliant: true`
    - Test that a token set with one failing color pair produces `compliant: false` and the report lists the failing pair with token names, actual ratio, and required minimum
    - Test that light and dark schemes are validated independently
    - _Requirements: 8.3, 8.4, 8.5_

- [ ] 10. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [ ] 11. Implement the brand registry
  - [ ] 11.1 Implement `registerBrand(registry: BrandRegistry, entry: BrandRegistryEntry): BrandRegistry` in `src/brands/registry.ts`
    - Add the entry with `status: "pending-validation"`
    - _Requirements: 9.1, 9.2_

  - [ ] 11.2 Implement `activateBrand(registry: BrandRegistry, brandId: string, tokenSet: TokenSet): { registry: BrandRegistry; report: ValidationReport }` in `src/brands/registry.ts`
    - Run `validateTokenSet` on the token set
    - If compliant, update the brand status to `"active"` and record `lastValidated`
    - If non-compliant, update the brand status to `"non-compliant"`
    - _Requirements: 9.3_

  - [ ] 11.3 Implement `findMissingTokens(registry: BrandRegistry, newSchema: typeof TOKEN_SCHEMA): Record<string, string[]>` in `src/brands/registry.ts`
    - For each active brand, identify token IDs in the new schema that are absent from the brand's token set
    - Return a map of `brandId → missingTokenIds[]`
    - _Requirements: 9.5_

  - [ ] 11.4 Implement independent versioning: `bumpTokenSetVersion(registry: BrandRegistry, brandId: string, newVersion: string): BrandRegistry`
    - Update only the specified brand's version; other brands are unchanged
    - _Requirements: 9.4_

  - [ ]* 11.5 Write property test for schema gap detection (Property 12)
    - **Property 12: Schema update gap detection**
    - **Validates: Requirements 9.5**
    - For any registry and any new required token ID not present in some brands, assert `findMissingTokens` returns exactly those brand IDs

  - [ ]* 11.6 Write unit tests for the brand registry
    - Test that registering a brand sets status to `"pending-validation"`
    - Test that activating a compliant brand sets status to `"active"`
    - Test that activating a non-compliant brand sets status to `"non-compliant"` and does not publish
    - Test that bumping one brand's version does not affect other brands
    - _Requirements: 9.2, 9.3, 9.4_

- [ ] 12. Implement the Figma Variables sync pipeline
  - [ ] 12.1 Implement `tokenIdToFigmaName(tokenId: string, level: "semantic" | "component"): string` in `src/figma/mapping.ts`
    - Semantic: `{category}/{semantic-name}` (e.g. `color/action/primary`)
    - Component: `{category}/{component}/{role}` (e.g. `color/button/background-default`)
    - _Requirements: 7.4_

  - [ ]* 12.2 Write property test for Figma naming convention (Property 11)
    - **Property 11: Figma variable naming convention**
    - **Validates: Requirements 7.4**
    - For any token ID, assert the generated Figma name matches the expected pattern

  - [ ] 12.3 Implement `buildFigmaPayload(registry: BrandRegistry, tokenSets: Record<string, TokenSet>): FigmaVariablesPayload` in `src/figma/sync.ts`
    - Group tokens into collections by category
    - Create one `FigmaMode` per brand per color scheme
    - Populate `valuesByMode` with resolved token values
    - _Requirements: 7.1, 7.2, 7.3_

  - [ ] 12.4 Implement `generateTokenMapping(tokenSet: TokenSet): TokenMappingEntry[]` in `src/figma/mapping.ts`
    - Return an array of `{ tokenId, figmaVariableName }` for all tokens
    - This serves as the traceability mapping document
    - _Requirements: 7.5_

  - [ ]* 12.5 Write unit tests for the Figma sync pipeline
    - Test that `buildFigmaPayload` produces one collection per token category
    - Test that each active brand has a corresponding mode in each collection
    - Test that updating a base token value changes the resolved value in the payload
    - _Requirements: 7.1, 7.2, 7.3_

- [ ] 13. Implement CSS and ESM token exporters
  - [ ] 13.1 Implement `exportToCss(tokenSet: TokenSet, brandId: string): string` in `src/export/css.ts`
    - Generate `:root` block with CSS custom properties for all tokens
    - Generate `[data-brand="{brandId}"][data-scheme="dark"]` block for dark scheme overrides
    - Token names use kebab-case: `--{category}-{semantic-name}`
    - _Requirements: 2.1, 3.7_

  - [ ] 13.2 Implement `exportToEsm(tokenSet: TokenSet): string` in `src/export/esm.ts`
    - Generate a TypeScript `const tokens = { ... } as const` object
    - Values reference CSS custom property names (e.g. `"var(--color-action-primary)"`)
    - _Requirements: 10.5_

  - [ ]* 13.3 Write unit tests for exporters
    - Test that CSS output contains `:root` block and dark scheme block
    - Test that all required semantic token IDs appear in the CSS output
    - Test that ESM output is valid TypeScript with `as const`
    - _Requirements: 3.7, 10.5_

- [ ] 14. Implement spacing base unit property test
  - [ ]* 14.1 Write property test for spacing scale base unit (Property 10)
    - **Property 10: Spacing scale base unit**
    - **Validates: Requirements 5.2**
    - For every spacing base token with step index N, assert its rem value equals `N × 0.25`

- [ ] 15. Wire everything together in the public API
  - [ ] 15.1 Update `src/index.ts` to export all public functions and types
    - Export: all TypeScript interfaces and types
    - Export: `validateSchema`, `validateReferenceIntegrity`, `validateTokenSet`, `canPublish`
    - Export: `extendTokenSet`, `diffTokenSets`, `registerBrand`, `activateBrand`, `findMissingTokens`
    - Export: `buildFigmaPayload`, `generateTokenMapping`
    - Export: `exportToCss`, `exportToEsm`
    - Export: `DYN_SPORTS_TOKEN_SET`, `TOKEN_SCHEMA`
    - _Requirements: 9.1_

  - [ ] 15.2 Register the DYN Sports brand in the default registry export
    - Create `src/brands/registry.ts` default export with DYN Sports pre-registered and activated
    - _Requirements: 9.2, 9.3_

  - [ ]* 15.3 Write integration tests for the end-to-end brand onboarding flow
    - Create a minimal new brand token set extending the default
    - Register it, activate it, assert it becomes `"active"` if compliant
    - Assert the Figma payload includes the new brand's modes
    - Assert the CSS export includes the new brand's custom properties
    - _Requirements: 9.1, 9.2, 9.3_

- [ ] 16. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Each task references specific requirements for traceability
- Checkpoints at tasks 4, 10, and 16 ensure incremental validation
- Property tests validate universal correctness properties (Properties 1–12 from the design document)
- Unit tests validate specific examples and edge cases
- The DYN Sports token set is the reference implementation; all other brands extend or follow the same pattern
