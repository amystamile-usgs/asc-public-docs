
# Automatic Registration

## Introduction

Automatic registration is the attempt to match a pattern in an image
cube. Pattern matching has many different purposes.

Examples:

1.  Register an entire image to a second image. The registration of an
    image is performed by moving (rubber-sheet) the pixels to output
    pixel locations that result in pattern matching directly to a second
    'fixed' or 'truth' image.

    - Use: [coreg](https://isis.astrogeology.usgs.gov/Application/presentation/Tabbed/coreg/coreg.html)

2.  Relative cartographic registration between a number of overlapping
    images. Pattern matching is used to build a control point network
    across a number of overlapping images for solving and adjusting the
    camera pointing that are then applied to map project the images.

    - Use: [pointreg](http://isis.astrogeology.usgs.gov/Application/presentation/Tabbed/pointreg/pointreg.html) or [jigsaw](http://isis.astrogeology.usgs.gov/Application/presentation/Tabbed/jigsaw/jigsaw.html)

3.  Find and record the pixel location(s) of a camera reference mark
    (e.g., Vidicon reseau mark) across an image

    - Use: [findrx](http://isis.astrogeology.usgs.gov/Application/presentation/Tabbed/findrx/findrx.html)


## Match Algorithm Definition Files (PVL)

Registration of images can be an advanced and complicated process. ISIS3
is designed to supply a choice of match/registration algorithms to
handle a variety of image data conditions. An ISIS3 "plugin" algorithm
expects all match algorithms to supply parameter information through a
Parameter Value Language (PVL) definition file. Examples of match
algorithm PVL's can be found in

    $ISIS3DATA/base/templates/autoreg/coreg*.def 

Registration applications (such as coreg) will allow the user to choose
from this collection of PVL files in the user interface gui.

A PVL file can be generated for a selected match algorithm with 
[autoregtemplate](http://isis.astrogeology.usgs.gov/Application/presentation/Tabbed/autoregtemplate/autoregtemplate.html).

## Match Algorithms

### Maximum Correlation

The Maximum Correlation match algorithm is most commonly used. The
algorithm computes a  
correlation coefficient that will range from no correlation (zero) to
perfect correlation (one).

Example templates for Maximum Correlation can be found in:

 
    $ISIS3DATA/base/templates/autoreg/coreg.maxcor.p*.s*.def.  


An example PVL for the Maximum Correlation follows:

    Object = AutoRegistration
    
      Group = Algorithm
        Name = MaximumCorrelation
        Tolerance = 0.7
      End_Group
    
    End_Object

!!! tip

    A successful correlation is met if the **Goodness of Fit** is **Greater Than** 
    the Tolerance parameter value for the Maximum Correlation Algorithm.

### Minimum Difference

The Minimum Difference match algorithm performs a subtraction of the
Pattern Chip and the sub-region of the Search Chip. The **Goodness of 
Fit** will range from zero for a perfect match, while larger values 
indicate a less likely match. The **Goodness of Fit** is measured as an 
absolute value and will never be negative.

Example templates for Minimum Difference can be found in:

    $ISIS3DATA/base/templates/autoreg/coreg.mindif.p*.s*.def.

An example PVL for the Minimum Difference follows:

    Object = AutoRegistration
    
      Group = Algorithm
        Name = MinimumDifference
        Tolerance = 100
      End_Group
    
    End_Object

!!! tip

    A successful correlation is met if the **Goodness of Fit** is **Less
    Than** the Tolerance parameter value for the Minimum Difference Algorithm.

## Tolerance

The Tolerance parameter is used by both Maximum
Correlation and Minimum
Difference. Each correlation coefficient result
(**Goodness of Fit**) is tested against the Tolerance. Tolerance is set
within the PVL file along with the chosen
registration algorithm.

Points that fail the Tolerance test are reported by the autoreg
applications as:

- `FitChipToleranceNotMet`

- `SurfaceModelToleranceNotMet`

## Chips

There are two chips used in automatic registration, the Pattern
Chip and the Search Chip. The
two chips are nothing more than sub-areas of the input image cubes,
generally small in size. An NxM Chip is defined to be an N-Sample by
M-Line region of an image. Basic elements of a chip are:

  - N and M are natural numbers (1, 2, 3...,50)
  - Like image cubes, chip coordinates are sample/line and 1-band based
  - The center of the chips is (N - 1)/2 + 1 and (M - 1)/2 + 1

### Pattern Chip

A Pattern Chip will contain the data you would like to match. This
Pattern Chip is basically extracted from the 'Reference' image (it is
also referred to as the 'Truth' image). The Pattern Chip is then walked
across the Search Chip that has been extracted from
the other image that is to be modified.

The PVL for a Pattern Chip defined as 25 Samples x 25 Lines (625 pixels)
is:

    Object = AutoRegistration
    
      Group = PatternChip
        Samples = 25
        Lines = 25
      End_Group
    
    End_Object

**Pattern Chip Size Requirement:**

N + M \>= 3. This ensures that the pattern is not a single pixel.

???+ tip "Tips"

    1.  The feature to match included in the Pattern Chip does not have to
        be in the center.
    2.  The more "bland" the data is within the input images, the larger the
        Pattern Chip box should be. A larger box will allow for more pixel
        DN variance.
    3.  In general, avoid very small box sizes less than 11 Samples X 11
        Lines
    4.  The smaller the size of the Pattern Chip, the greater the chance of
        matching too many areas in the search chip and returning a 'false
        positive' match.

### Search Chip

A Search Chip is the sub-area of the image cube that the pattern might
be found in. The Pattern Chip is walked through the
Search Chip looking for the best match. The Search Chip must be larger
than the Pattern Chip (at least one pixel bigger on line/sample size).

The PVL for a Search Chip defined as 45 Samples x 45 Lines (2025 pixels)
is:

    Object = AutoRegistration
    
      Group = PatternChip
        Samples = 25
        Lines = 25
      End_Group
    
      Group = SearchChip
        Samples = 45
        Lines = 45
      End_Group
    
    End_Object

**Search Chip Size Requirement:**

N(search) \>= N(pattern) + 2 and similarly for M. This ensures that the
Pattern Chip spans at least a 3x3 window within the Search Chip. An
important requirement for surface fitting in order to compute sub-pixel
accuracy. (Refer to: [Sub-Pixel Positioning](../isis-fundamentals/cube-format.md#sub-pixel-positioning)).

???+ tip "Tips"

    - The Search Chip must be larger than the Pattern Chip (at least one
        pixel bigger on line/sample size).
    - The Search Chip needs to be large enough to allow a 'best guess' or
        predicted line and sample mis-registration offset between the images
        that are to be correlated.  
        - **SearchChipSize = PatternChipSize + (OffsetPixelEstimate *
            2)**
    - BUT, the Search Chip shouldn't be too much larger than the Pattern
        Chip.  
        - The difference in size could greatly affect how long the
            registration application would run.
    - Refer to PVL example above:
        - An estimated offset between two input images of 10 Lines by 10
            Samples will need a Search Chip size (10 Line offset * 2) by
            (10 Sample offset * 2). The Search Chip size in this example
            needs to be at least 20*20 Lines/Samples bigger than the pattern
            chip.
        - PatternChipSize = 25 * 25
        - SearchLineDimention = 25 + (10 * 2) = 45 * SearchSampleDimention = 25 + (10 * 2) = 45
        - SearchChipSize= 45 * 45

## Restricting Input Pixel Ranges

The validity of the [input pixels](../isis-fundamentals/cube-format.md#what-are-pixels) 
of the Pattern Chip is the very first test
performed. Prior to the match algorithm being invoked during the walk
process, a simple test is performed to ensure there are enough pixels to
work with. Pixels are deemed valid if they are in the minimum/maximum
range and/or they are not [special pixel](../isis-fundamentals/special-pixels.md) values.

The Pattern Chip is only checked once. If it does not contain enough
valid pixels, the match is deemed to fail. Otherwise, the walk through
occurs and the sub-area is extracted from the Search
Chip. If this sub-area does not have enough valid
pixels a match will be deemed to fail at that search location.

Points that fail the default or specified Valid Minimum/Maximum and/or
ValidPercent tests are reported by the autoreg applications as:

- `PatternNotEnoughValidData`

- `FitChipNoData`

- `SurfaceModelToleranceNotMet`

### Valid Minimum/Maximum

Input image pixels (DN) values may be excluded from the match
algorithm if they fall outside of a specific range. The ranges can be
specified independently for the Search Chip and Pattern Chip. While this
might not be necessary for a successful registration, excluding pixel
data could improve the chances depending on the characteristics of the
input images. (i.e., Eliminating the DN range of deep shadow areas in an
image).

The ranges are handled via the PVL as follows:

    Object = AutoRegistration
    
      Group = PatternChip
        Samples = 20
        Lines = 20
        ValidMinimum = 0.1
        ValidMaximum = 0.4
      End_Group
    
      Group = SearchChip
        Samples = 55
        Lines = 55
        ValidMinimum = 2.5
        ValidMaximum = 10.5
      End_Group
    
    End_Group

### Valid Pixel Count (ValidPercent)

`ValidPercent` defines the percentage of valid data that is required
within the Pattern Chip or Search Chip. The parameter can be set in
conjunction with Valid Minimum/Maximum.

The Valid Percent would be handled via the PVL as follows:

    Object = AutoRegistration
    
      Group = PatternChip
        Samples = 20
        Lines = 20
        ValidPercent = 80
      End_Group 
    End_Group

Default for `ValidPercent`: `AutoRegDefaults`

!!! tip

    `ValidPercent` is specified in the Pattern Chip, as the sub-area 
    chip of the search area will use the same value.

## Fit Chip

As the Pattern Chip walks through the sub-regions
of the Search Chip , a third chip is generated
called a Fit Chip. The Fit Chip will have the same line and sample
dimensions as the Search Chip. The Fit Chip is filled with the resulting
**Goodness of Fit** values at every position that the Pattern Chip is walked
across the Search Chip computing a fit correlation value.

The highest or lowest **Goodness of Fit** values within the Fit Chip
generally represent the position that best matched between the Pattern
and Search areas. It is only good to a maximum of **ONE pixel accuracy**.

The Fit Chip can be interactively generated and viewed using the image
display application  
[qnet](http://isis.astrogeology.usgs.gov/Application/presentation/Tabbed/qnet/qnet.html).


## Sub-Pixel Accuracy

The attempt to reach a registration of sub-pixel accuracy is performed
on the Fit Chip. The Fit Chip is created as the
Pattern Chip is walked through Search
Chip. The Fit Chip contains the resulting
correlation position between the two chips of a maximum of ONE pixel
accuracy. In many cases, the actual, ultimate registration may lie
somewhere between two pixels. (Refer to:
[Sub-Pixel Positioning](../isis-fundamentals/cube-format.md#sub-pixel-positioning)).

The sub-pixel accuracy can be turned off through the PVL settings as
follows:

    Group = Algorithm
      Name = MaximumCorrelation
      SubPixelAccuracy = False
    End_Group

When the sub-pixel accuracy is turned off, the whole pixel with the best
fit is returned.  
Default for SubPixelAccuracy: AutoRegDefaults

!!! tip

    By default, if an ideal **Goodness of Fit** is found (e.g. Zero (0.0) for
    Minimum Difference or One (1.0) for Maximum Correlation), we have a perfect 
    fit and assume that it is the best position. In this case, the sub-pixel 
    accuracy phase is omitted.

### Surface Modeling

A continuous mathematical surface is modeled based on the data within
the Fit Chip. A 2nd degree 2-dimensional
polynomial is calculated given an NxN window of points extracted from
the Fit Chip surrounding the correlation peak (highest value)-which
represents the whole pixel registration. An estimate of the true
sub-pixel registration position of the chip can be reached based on this
surface model.

A perfect 3D linear fit (global maximum) would be represented by a
spherical surface with a single high peak.

## AutoRegDefaults

(As of May 20, 2009)

    ValidPercent                  = 50.0
    MinimumZScore                 = 1.0
    Tolerance                     = Null (no default)
    ReductionFactor               = 1.0
    
    SubPixelAccuracy              = True
    SurfaceModelDistanceTolerance = 1.5
    SurfaceModelWindows           = 5
    SurfaceModelEccentricityRatio = 2 (2:1)
