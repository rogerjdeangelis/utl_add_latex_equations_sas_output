# utl_add_latex_equations_sas_output
Adding LaTeX equations to SAS pdf and powerpoint tables and graphs

    Adding Latex mathematical equations to SAS output pdf and pptx

     You need to install MikTeX (for LaTeX), ghostscript, Pyhton, BocOft, and MS Powerpoint

     Final PDF and PPT
      https://www.dropbox.com/s/vldvbo3zc7s43p2/outputs.pdf?dl=0
      https://www.dropbox.com/s/7tmtq2bmd1z4yu2/outputs.ppt?dl=0

     Inputs and intermediate files
       Equation
         https://www.dropbox.com/s/y6jdp3u0moqtc9f/equ.pdf?dl=0
       SAS Table
         https://www.dropbox.com/s/n3fixqi0vd97v4o/abc.pdf?dl=0
       Cropped Equation (could make this a png using python package wand)
         https://www.dropbox.com/s/uxpiwuc0olcuyey/cropped.pdf?dl=0


     Not happy with this post (need to automate especially cropping of LaTeX output pdf)

        WORKING CODE

           Create formula for roots of quadratic usin LaTeX

             d:/tex/equ.tex

               \documentclass[landscape]{article}
               \setlength{\pdfpagewidth}{\paperwidth}
               \setlength{\pdfpageheight}{\paperheight}
               \begin{document}
               $$ \mbox{\Huge $ {x = \frac{ { - b \pm \sqrt {b^2 - 4ac} } }{2a}}$ } $$
               \end{document}

           Create the pdf with the equation

               x 'pdflatex -output-directory d:/pdf d:/tex/equ.tex';

           Crop the page isolating the equation;

               x 'd:/pdf/gswin64c.exe -o d:/pdf/cropped.pdf -sDEVICE=pdfwrite
                 -c "[/CropBox [250 520 533 400]" -c " /PAGES pdfmark" -f d:/pdf/equ.pdf';

           Create SAS pdf output table or graph

               ods pdf file="d:/pdf/abc.pdf";
               proc report data=have nowd style(header)={font_size=16pt}
                    style(column)={font_size=16pt};
               cols Polynomial a b c;
               define Polynomial / display;
               define a / display;

           Overlay the two pdfs (Python)

               input = PdfFileReader(open("d:/pdf/abc.pdf", "rb"));
               watermark = PdfFileReader(open("d:/pdf/cropped.pdf", "rb"));
               page = input.getPage(0);
               page.mergePage(watermark.getPage(0));
               outputStream = file("d:/pdf/outputs.pdf", "wb");

           Make a Powerpoint slide

               x "d:\exe\p2p\pdftopptcmd.exe d:\pdf\outputs.pdf d:\ppt\outputs.ppt";


    HAVE
    ====

     Up to 40 obs from have total obs=12

     Obs    POLYNOMIAL    A     B    C    ROOT

       1    Quadratic     1    -2    2      -2
       2    Quadratic     1    -2    4      -6
       3    Quadratic     1    -2    9     -16
       4    Quadratic     1    -4    2       4
       5    Quadratic     1    -4    4       0
       6    Quadratic     1    -4    9     -10
       7    Quadratic     2    -2    2      -3
       8    Quadratic     2    -2    4      -7
       9    Quadratic     2    -2    9     -17
      10    Quadratic     2    -4    2       0
      11    Quadratic     2    -4    4      -4
      12    Quadratic     2    -4    9     -14

    WANT
    ====

         POLYNOMIAL    A     B    C    ROOT
                                                 This is the output
         Quadratic     1    -2    2      -2
         Quadratic     1    -2    4      -6               _____________
         Quadratic     1    -2    9     -16              /
         Quadratic     1    -4    2       4             /   2
         Quadratic     1    -4    4       0      --    /   B    -  4AC
         Quadratic     1    -4    9     -10        \  /
         Quadratic     2    -2    2      -3     ____\/__________________
         Quadratic     2    -2    4      -7
         Quadratic     2    -2    9     -17                  2A
         Quadratic     2    -4    2       0
         Quadratic     2    -4    4      -4
         Quadratic     2    -4    9     -14


    *                _              _       _
     _ __ ___   __ _| | _____    __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \  / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/ | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|  \__,_|\__,_|\__\__,_|

    ;

    title;
    footnote;
    options orientation=landscape;

    *just in case;
    %utlfkil(d:/pdf/abc.pdf);
    %utlfkil(d:/pdf/equ.pdf);
    %utlfkil(d:/pdf/cropped.pdf);
    %utlfkil(d:/pdf/outputs.pdf);
    %utlfkil(d:/pdf/outputs.ppt);
    %utlfkil(d:/tes/equ.tex);

    proc datasets lib=work kill;
    run;quit;

    data _null_;
     file "d:/tex/equ.tex";
     input;
     put _infile_;
    cards4;
    \documentclass[landscape]{article}
    \setlength{\pdfpagewidth}{\paperwidth}
    \setlength{\pdfpageheight}{\paperheight}
    \begin{document}
    $$ \mbox{\Huge $ {x = \frac{ { - b \pm \sqrt {b^2 - 4ac} } }{2a}}$ } $$
    \end{document}
    ;;;;
    run;quit;

    data have;
      retain Polynomial "Quadratic";
      do a=1 to 2;
       do b=-2 to -4 by -2;
         do c=2,4,9;
           root=round((b**2 - 4*a*c)/(2*a),.1);
           output;
         end;
       end;
      end;
    run;quit;

    *          _       _   _
     ___  ___ | |_   _| |_(_) ___  _ __
    / __|/ _ \| | | | | __| |/ _ \| '_ \
    \__ \ (_) | | |_| | |_| | (_) | | | |
    |___/\___/|_|\__,_|\__|_|\___/|_| |_|

    ;

    ods pdf file="d:/pdf/abc.pdf";

    proc report data=have nowd style(header)={font_size=16pt}
         style(column)={font_size=16pt};
    cols Polynomial a b c;
    define Polynomial / display;
    define a / display;
    run;quit;

    ods pdf close;

    * create d:/pdf/equ.pdf (full page equation);
    x 'pdflatex -output-directory d:/pdf d:/tex/equ.tex';

    * crop the full page equation;
    * 72pts per inch the bigger the numbers in cropbox the smaller the area, a rough 1-2 aspect must be maintained 533/400=1.33?
      max box for 8.5 width to 11.5 is  [24 72 559 794];
    x 'd:/pdf/gswin64c.exe -o d:/pdf/cropped.pdf -sDEVICE=pdfwrite -c "[/CropBox [210 640 400 730]" -c " /PAGES pdfmark" -f d:/pdf/equ.pdf';

    %utl_submit_py64('
    from PyPDF2 import PdfFileWriter, PdfFileReader;
    output = PdfFileWriter();
    input = PdfFileReader(open("d:/pdf/abc.pdf", "rb"));
    watermark = PdfFileReader(open("d:/pdf/cropped.pdf", "rb"));
    page = input.getPage(0);
    page.mergePage(watermark.getPage(0));
    output.addPage(page);
    outputStream = file("d:/pdf/outputs.pdf", "wb");
    output.write(outputStream);
    outputStream.close();
    ');

    * to powerpoint;
    x "d:\exe\p2p\pdftopptcmd.exe d:\pdf\outputs.pdf d:\ppt\outputs.ppt";





