# Test: FontMake, a Glyphs source, a Width axis, and naming issues

Can FontMake produce accurate static OTFs and variable TTFs, with a width axis, from a single Glyphs source?

I’ve attempted to set up a GlyphsApp source with:
- A Weight axis
- A Width axis
- Exports/instances that will output different sub families in static fonts – one per main width location
- Export/instance data that will produce desired instance names in a variable font

Main problem:
- If I set Glyphs data to produce accurate variable TTF instance naming, like `Condensed Medium`, the CFF table of OTFs gets duplicated width desriptors, like `FamilynameCondensed-CondensedMedium`
- If I set Glyphs data to procue accurate CFF table naming, like `FamilynameCondensed-Medium`, the variable font gets incorrect instance naming, with two instances called simply `Medium`

FontMake (or GlyphsLib?) problems:
1. The `Localized Family Name` and `Localized Style Name` properties of Glyphs Exports are sometimes ignored in building fonts.
2. `name table entry` Custom Properties are ignored when building names in the CFF table of OTF fonts.

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
