# Test: Can FontMake build from Glyphs to multiple families, static & variable? If not, what is blocking it?

[WIP test case for a planned glyphsLib issue]

An earlier, partial issue was filed at [FontMake Issue #1108](https://github.com/googlefonts/fontmake/issues/1108), but this intends to clarify that issue, ultimately as a step towards solving it.

---

## Context and Goals

There are two basic ways of organizing Width + Weight families:
1. A single family, containing all combinations of Width & Weight locations. Example: There is a single [Noto Sans](https://github.com/notofonts/latin-greek-cyrillic) family which contains styles like `Condensed Medium` and `Expanded Light`. It has this organization of styles for both the variable and static fonts. This approach is typical of Google Fonts projects. In the Google Fonts approach, there is no distinction in family naming between variable and static fonts. The argument here is that it keeps things simple, in a way, as it reveals no distinction between static and variable fonts.
2. Multiple families, split up by widths, each with weight styles. Example: [Frere-Jones Mallory](https://frerejones.com/families/mallory) includes subfamilies like `Mallory Condensed` and `Mallory Extended`, which in turn include styles like `Medium` and `Light`. This approach is typical of retail foundries, including many families listed on Adobe Fonts. Often, the variable font in such a family is given its own, separate family, e.g. `Mallory Variable`, which itself contains all styles, such as `Condensed Medium` and `Expanded Light`. In reality, these “subfamilies” are just different subdivisions of a single superfamily, but they can be a helpful way to give expert users easier access to the fonts and styles they want, especially in contexts where variable font features may be desired, or static font reliability may be preferred.

The first approach is very simple to build with FontMake, from a Glyphs source with minimal configuration.

The second approach is seemingly not possible to build with FontMake, from a Glyphs source, at the time being, even with the use of configurations that make such a build possible directly from Glyphs.

- [ ] TODO: determine if Approach 2 is possible to build with FontMake, from a single `.designspace` file and UFO sources.

So, the goal of this repo is to identify what, exactly, is preventing the second build from working.


## Build Objective

The objective is to use FontMake to build from a single Glyphs source to the following fonts:

- A Variable font with all Width + Weight instances as styles in the `fvar` table, like:
    - Familyname Variable 
      - Condensed Regular
      - Condensed Medium
      - Condensed Bold
      – Regular
      - Medium
      – Bold
      – Wide Regular
      – Wide Medium
      – Wide Bold
- Static fonts which are split into Width-based subfamilies, with weights inside, like:
    - Familyname Condensed 
      – Regular
      - Medium
      – Bold
    - Familyname [Normal] 
      – Regular
      - Medium
      – Bold
    - Familyname Wide
      – Regular
      - Medium
      – Bold

## Tests

Can FontMake produce accurate static OTFs and variable TTFs, with a width axis, from a single Glyphs source?

I’ve attempted to set up a GlyphsApp source with:
- A Weight axis
- A Width axis
- Instances that will output different sub-families in static fonts – one per main width location, like `Familyname` and `Familyname Condensed`, as is typical for weight+width superfamilies.
- Instances that will produce desired instance names in a variable font, including width descriptors

Main problem:
- If I set Glyphs data to produce accurate variable TTF instance naming, like `Condensed Medium`, the CFF table of OTFs gets duplicated width desriptors, like `FamilynameCondensed-CondensedMedium`
- If I set Glyphs data to procue accurate CFF table naming, like `FamilynameCondensed-Medium`, the variable font gets incorrect instance naming, with two instances called simply `Medium`

Problems:
1. The `Localized Family Name` and `Localized Style Name` properties of Glyphs Exports are sometimes (maybe always?) ignored in building fonts.

Versions:
- FontMake 3.9.0 (also occurs with 3.8.1)
- GlyphsApp 3.2.3 (3260)

## Reproduction

Set up a venv if you want, and install FontMake.

### Test 1: Basic problem

Build the Glyphs font with fontmake:

```bash
fontmake -o variable -g source/test-1-build-fontmake-glyphsapp.glyphs --output-dir fonts/test1
fontmake -o otf -i -g source/test-1-build-fontmake-glyphsapp.glyphs --output-dir fonts/test1/otf
```

Then, check the variable font’s `fvar` table:

```bash
ttx -t fvar -o- fonts/test1/Familyname-VF.ttf
```

This looks good!

Then, check the `CFF` and `name` tables with TTX:

```bash
ttx -t name -t CFF 'fonts/test2/otf/FamilynameCondensed-Medium.otf'
```

This yields some obviously wrong results. Here are snippets:

```xml
    <namerecord nameID="1" platformID="3" platEncID="1" langID="0x409">
      Familyname Condensed Condensed Medium
    </namerecord>
    <namerecord nameID="2" platformID="3" platEncID="1" langID="0x409">
      Regular
    </namerecord>
    <namerecord nameID="3" platformID="3" platEncID="1" langID="0x409">
      1.000;NONE;FamilynameCondensed-CondensedMedium
    </namerecord>
    <namerecord nameID="4" platformID="3" platEncID="1" langID="0x409">
      Familyname Condensed Condensed Medium
    </namerecord>
```

```xml
    <CFFFont name="FamilynameCondensed-CondensedMedium">
      <version value="1.0"/>
      <FullName value="Familyname Condensed Condensed Medium"/>
      <FamilyName value="Familyname Condensed"/>
```


The `name` table records can be corrected with Custom Properties in the Glyphs source. However, these seem to have no impact on the CFF table. Let’s test that. 


### Test 2: the problem persists even with Custom Properties

I thought I could maybe solve this by adding name table ID-specific names to the Exports in my Glyphs source.

This probably shouldn’t be necessary, as Test 1 already includes `Localized Family Name` and `Localized Style Name` properties of Glyphs Exports. However, it is a natural next step, and it seems worth noting that it doesn’t work as expected.

Build the Glyphs font with fontmake:

```bash
fontmake -o variable -g source/test-2-build-fontmake-glyphsapp.glyphs --output-dir fonts/test2
fontmake -o otf -i -g source/test-2-build-fontmake-glyphsapp.glyphs --output-dir fonts/test2/otf
```

Then, check the variable font’s `fvar` table:

```bash
ttx -t fvar -o- fonts/test2/Familyname-VF.ttf
```

This looks good!

Then, check the `CFF` and `name` tables with TTX:

```bash
ttx -t name -t CFF 'fonts/test2/otf/FamilynameCondensed-Medium.otf'
```

This yields correct NameIDs 1–4, but the CFF table is still incorrect. Here are snippets:

```xml
    <namerecord nameID="1" platformID="3" platEncID="1" langID="0x409">
      Familyname Condensed Medium
    </namerecord>
    <namerecord nameID="2" platformID="3" platEncID="1" langID="0x409">
      Regular
    </namerecord>
    <namerecord nameID="3" platformID="3" platEncID="1" langID="0x409">
      1.000;NONE;FamilynameCondensed-Medium
    </namerecord>
    <namerecord nameID="4" platformID="3" platEncID="1" langID="0x409">
      Familyname Condensed Medium
    </namerecord>
```

```xml
    <CFFFont name="FamilynameCondensed-CondensedMedium">
      <version value="1.0"/>
      <FullName value="Familyname Condensed Condensed Medium"/>
      <FamilyName value="Familyname Condensed"/>
```

### Test 3: Correcting static names messes up variable instance names

What if we simplify the basic names of Exports in Glyphs? It works for static fonts, but not the variable font.

Build:

```bash
fontmake -o variable -g source/test-3-build-fontmake-glyphsapp.glyphs --output-dir fonts/test3
fontmake -o otf -i -g source/test-3-build-fontmake-glyphsapp.glyphs --output-dir fonts/test3/otf
```

Let’s check the `CFF` and `name` tables with TTX:

```bash
ttx -t name -t CFF 'fonts/test2/otf/FamilynameCondensed-Medium.otf'
```

These both look good:

```xml
    <namerecord nameID="1" platformID="3" platEncID="1" langID="0x409">
      Familyname Condensed Medium
    </namerecord>
    <namerecord nameID="2" platformID="3" platEncID="1" langID="0x409">
      Regular
    </namerecord>
    <namerecord nameID="3" platformID="3" platEncID="1" langID="0x409">
      1.000;NONE;FamilynameCondensed-Medium
    </namerecord>
    <namerecord nameID="4" platformID="3" platEncID="1" langID="0x409">
      Familyname Condensed Medium
    </namerecord>
```

```xml
    <CFFFont name="FamilynameCondensed-Medium">
      <version value="1.0"/>
      <FullName value="Familyname Condensed Medium"/>
      <FamilyName value="Familyname Condensed"/>
```

However, the variable font no longer has correct instance names:

```bash
ttx -t fvar -o- fonts/test3/Familyname-VF.ttf
```

...because now, the instance names repeat for Condensed and Normal styles:

```xml
    <!-- Regular -->
    <NamedInstance flags="0x0" subfamilyNameID="258">
      <coord axis="wght" value="400.0"/>
      <coord axis="wdth" value="50.0"/>
    </NamedInstance>

    <!-- Medium -->
    <NamedInstance flags="0x0" subfamilyNameID="259">
      <coord axis="wght" value="500.0"/>
      <coord axis="wdth" value="50.0"/>
    </NamedInstance>

    <!-- Bold -->
    <NamedInstance flags="0x0" subfamilyNameID="260">
      <coord axis="wght" value="700.0"/>
      <coord axis="wdth" value="50.0"/>
    </NamedInstance>

    <!-- Regular -->
    <NamedInstance flags="0x0" subfamilyNameID="258">
      <coord axis="wght" value="400.0"/>
      <coord axis="wdth" value="100.0"/>
    </NamedInstance>

    <!-- Medium -->
    <NamedInstance flags="0x0" subfamilyNameID="259">
      <coord axis="wght" value="500.0"/>
      <coord axis="wdth" value="100.0"/>
    </NamedInstance>

    <!-- Bold -->
    <NamedInstance flags="0x0" subfamilyNameID="260">
      <coord axis="wght" value="700.0"/>
      <coord axis="wdth" value="100.0"/>
    </NamedInstance>
```

## Conclusion

I might be doing something wrong, but after a lot of experimentation, it certainly seems like an issue in FontMake or GlyphsLib.

Any insights are appreciated!
