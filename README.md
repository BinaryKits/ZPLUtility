# ZPLUtility
A .net library helping to generate ZPL string.
Please refer to the Programming Guide for raw ZPL code definitaion, <s>https://www.zebra.com/content/dam/zebra/manuals/en-us/software/zpl-zbi2-pm-en.pdf</s>

[https://github.com/robmachado/webetiq/blob/master/local/Doc/zpl-zbi2-pm-en.pdf](https://github.com/robmachado/webetiq/blob/master/local/Doc/zpl-zbi2-pm-en.pdf)

Some basic ZPL elements are included, if you have any suggestions please feel free to let me know.

[![NuGet-Stable](https://img.shields.io/nuget/v/ZPLUtility.svg?label=NuGet%20stable)](https://www.nuget.org/packages/ZPLUtility/)

You can test generated ZPL code at http://labelary.com/viewer.html

## Usage:
### Single element
```C#
var result = new ZPLGraphicBox(100, 100, 100, 100).ToZPLString();
Console.WriteLine(result); 
//Output
//^FO100,100
//^GB100,100,1,B,0^FS
```
### Barcode
```C#
var result = new ZPLBarCode128("123ABC", 100, 300).ToZPLString();
Console.WriteLine(result);
//Output
//^FO100,300
//^BCN,100,Y,N
//^FD123ABC^FS
```
### Whole label
```C#
var sampleText = "[_~^][LineBreak\n][The quick fox jumps over the lazy dog.]";
ZPLFont font = new ZPLFont(fontWidth: 50, fontHeight: 50);
var labelElements = new List<ZPLElementBase>();
labelElements.Add(new ZPLTextField(sampleText, 50, 100, font));
labelElements.Add(new ZPLGraphicBox(400, 700, 100, 100, 5));
labelElements.Add(new ZPLGraphicBox(450, 750, 100, 100, 50, ZPLConstants.LineColor.White));
labelElements.Add(new ZPLGraphicCircle(400, 700, 100, 5));
labelElements.Add(new ZPLGraphicDiagonalLine(400, 700, 100, 50, 5));
labelElements.Add(new ZPLGraphicDiagonalLine(400, 700, 50, 100, 5));
labelElements.Add(new ZPLGraphicSymbol(ZPLGraphicSymbol.GraphicSymbolCharacter.Copyright, 600, 600, 50, 50));

//Add raw ZPL code
labelElements.Add(new ZPLRaw("^FO200, 200^GB300, 200, 10 ^FS"));

var renderEngine = new ZPLEngine(labelElements);
var output = renderEngine.ToZPLString(new ZPLRenderOptions() { AddEmptyLineBeforeElementStart = true });

Console.WriteLine(output);
```
### Simple layout
```C#
var elements = new List<ZPLElementBase>();

var o = new ZPLOrigin(100, 100);
for (int i = 0; i < 3; i++)
{
    for (int j = 0; j < 3; j++)
    {
        elements.Add(new ZPLGraphicBox(o.PositionX, o.PositionY, 50, 50));
        o = o.Offset(0, 100);
    }
    o = o.Offset(100, -300);
}

var options = new ZPLRenderOptions();
var output = new ZPLEngine(elements).ToZPLString(options);

Console.WriteLine(output);
```
### Auto scale based on DPI
```C#
var labelElements = new List<ZPLElementBase>();
labelElements.Add(new ZPLGraphicBox(400, 700, 100, 100, 5));

var options = new ZPLRenderOptions() { SourcePrintDPI = 203, TargetPrintDPI = 300 };
var output = new ZPLEngine(labelElements).ToZPLString(options);

Console.WriteLine(output);
```
### Render with comment for easy debugging
```C#
var labelElements = new List<ZPLElementBase>();

var textField = new ZPLTextField("AAA", 50, 100, ZPLConstants.Font.Default);
textField.Comments.Add("An important field");
labelElements.Add(textField);

var renderEngine = new ZPLEngine(labelElements);
var output = renderEngine.ToZPLString(new ZPLRenderOptions() { DisplayComments = true });

Console.WriteLine(output);
```

### Different text field type
```C#
var sampleText = "[_~^][LineBreak\n][The quick fox jumps over the lazy dog.]";
ZPLFont font = new ZPLFont(fontWidth: 50, fontHeight: 50);

var labelElements = new List<ZPLElementBase>();
//Specail character is repalced with space
labelElements.Add(new ZPLTextField(sampleText, 10, 10, font, useHexadecimalIndicator: false));
//Specail character is repalced Hex value using ^FH
labelElements.Add(new ZPLTextField(sampleText, 10, 50, font, useHexadecimalIndicator: true));
//Only the first line is displayed
labelElements.Add(new ZPLSingleLineFieldBlock(sampleText, 10, 150, 500, font));
//Max 2 lines, text exceeding the maximum number of lines overwrites the last line.
labelElements.Add(new ZPLFieldBlock(sampleText, 10, 300, 400, font, 2));
// Multi - line text within a box region
labelElements.Add(new ZPLTextBlock(sampleText, 10, 600, 400, 100, font));

var renderEngine = new ZPLEngine(labelElements);
var output = renderEngine.ToZPLString(new ZPLRenderOptions() { AddEmptyLineBeforeElementStart = true });

Console.WriteLine(output);
```
### Draw pictures, auto resize based on DPI (Please dither the colorful picture first)
You have 2 options:
1. Use ~DY and ^IM
```C#
var elements = new List<ZPLElementBase>();
elements.Add(new ZPLGraphicBox(0, 0, 100, 100, 4));
elements.Add(new ZPLDownloadObjects('R', "SAMPLE.PNG", new System.Drawing.Bitmap("sample.bmp")));
elements.Add(new ZPLImageMove(100, 100, 'R', "SAMPLE", "PNG"));

var renderEngine = new ZPLEngine(elements);
var output = renderEngine.ToZPLString(new ZPLRenderOptions() { AddEmptyLineBeforeElementStart = true, TargetPrintDPI = 300, SourcePrintDPI = 200 });

Console.WriteLine(output);
```

2. Use ~DG and ^XG

```C#
var elements = new List<ZPLElementBase>();
elements.Add(new ZPLDownloadGraphics('R', "SAMPLE", "GRC", new System.Drawing.Bitmap("Sample.bmp")));
elements.Add(new ZPLRecallGraphic(100, 100, 'R', "SAMPLE", "GRC"));

var renderEngine = new ZPLEngine(elements);
var output = renderEngine.ToZPLString(new ZPLRenderOptions() { AddEmptyLineBeforeElementStart = true, TargetPrintDPI = 600, SourcePrintDPI = 200 });

Console.WriteLine(output);
```

