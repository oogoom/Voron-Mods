Files to make your slicer have a tabbed bed and west3d black PEI images for prusaslicer/superslicer and similar variants.

## IMPORTANT: 
The auto arrange algorithm in prusaslicer/superslicer is unable to handle custom shapes.  Devs seem to be aware of this issue but since late 2019, there hasn't been a fixed it seems.  If you only ever manually arrange parts and want a clean tabbed plate look, choose method 1.  If you like using auto arrange, choose method 2.

## METHOD 1
### Installation:
1) Go under printer settings, and click the SET button for Bed Shape.
2) Under SHAPE, select CUSTOM from the drop down menu.  Load the stl file for your bed size.  Choose the file with SHAPE in the name.
3) Under MODEL, Select and load the stl with the correct size for your printer.  Choose the file with MODEL in the name.
4) Under TEXTURE, click LOAD, and select the image file you want.
5) Click OK.

## METHOD 2
### Installation:
1) Go under printer settings, and click the SET button for Sed Shape.
2) Under SHAPE, select RECTANGULAR from the drop down menu.  Set your X and Y values for your bed (e.g., 350 size would use X: 350 and Y: 350).
3) Under MODEL, Select and load the stl with the correct size for your printer.  Choose the file with MODEL in the name.
4) Under TEXTURE, click LOAD, and select the image file you want.
5) Click OK.


Your 3D view should now have a tabbed sheet preview.

![Example](https://github.com/oogoom/Voron-Mods/blob/main/Slicer/sample.jpg)
