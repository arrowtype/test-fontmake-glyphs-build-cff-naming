# Test: FontMake, a Glyphs source, a Width axis, and naming issues

Can FontMake produce accurate static OTFs and variable TTFs, with a width axis, from a single Glyphs source?

I’ve attempted to set up a GlyphsApp source with:
- A Weight axis
- A Width axis
- Exports/instances that will output different sub families in static fonts – one per main width location
- Export/instance data that will produce desired instance names in a variable font

Problem:
- If I set Glyphs data to produce accurate variable TTF instance naming, like `Condensed Medium`, the CFF table of OTFs gets duplicated width desriptors, like `FamilynameCondensed-CondensedMedium`
- If I set Glyphs data to procue accurate CFF table naming, like `FamilynameCondensed-Medium`, the variable font gets incorrect instance naming, with two instances called simply `Medium`

Versions:
- FontMake 3.9.0 (also occurs with 3.8.1)
- GlyphsApp 3.2.3 (3260)

## Reproduction

Set up a venv if you want, and install FontMake.

### Test 1: Basic problem

Build the Glyphs font with fontmake:

```
fontmake -o variable -g source/test-build-fontmake-glyphsapp.glyphs --output-dir fonts
```

```
fontmake -o otf -g source/test-build-fontmake-glyphsapp.glyphs --output-dir fonts/otf
```


Then, check the variable font’s `fvar` table:

```
ttx -t fvar -o- fonts/Familyname-VF.ttf
```

This looks good!

Then, check a `CFF` and `name` tables with TTX:

```
ttx -t CFF 'fonts/otf/FamilynameCondensed-Medium.otf'
```

This yields some obviously wrong results. Here are snippets:

```
    <CFFFont name="FamilynameCondensed-CondensedMedium">
      <version value="1.0"/>
      <FullName value="Familyname Condensed Condensed Medium"/>
      <FamilyName value="Familyname Condensed"/>
```

```
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

The `name` table records can be corrected with Custom Properties in the Glyphs source. However, these seem to have no impact on the CFF table. 


### Test 2: the problem persists even with Custom Properties