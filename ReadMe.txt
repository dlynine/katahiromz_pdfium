Windows的PDFium封装类 by katahiromz.
增加支持XP系统的PDFium调用DLL
![image](http://github.com/dlynine/katahiromz_pdfium/blob/master/QQ截图20190524174326.jpg)
Cross compiler. Reduce the building time.

Usage:

    #include <windows.h>
    #include "katahiromz_pdfium.h"

    HBITMAP load_pdf_page(const char *file_name, int nPageIndex = 0) {
        katahiromz_pdfium  pdfium(true);
        if (!pdfium.is_loaded()) {
            return NULL;
        }

        // load a document
        HBITMAP hbm = NULL;
        FPDF_DOCUMENT pdf_doc = pdfium.FPDF_LoadDocument(file_name, NULL);
        if (pdf_doc != NULL) {
            // load a page
            FPDF_PAGE pdf_page = pdfium.FPDF_LoadPage(pdf_doc, nPageIndex);
            if (pdf_page != NULL) {
                // get the page size
                double page_width = pdfium.FPDF_GetPageWidth(pdf_page);
                double page_height = pdfium.FPDF_GetPageHeight(pdf_page);

                HDC hDC = ::GetDC(m_hWnd);
                SIZE sizPixels;

                // page size in pixels
                int logpixelsx = ::GetDeviceCaps(hDC, LOGPIXELSX);
                int logpixelsy = ::GetDeviceCaps(hDC, LOGPIXELSY);
                sizPixels.cx = long(page_width * logpixelsx / 72);
                sizPixels.cy = long(page_height * logpixelsy / 72);

                HDC hMemDC = ::CreateCompatibleDC(hDC);
                hbm = ::CreateCompatibleBitmap(hDC, sizPixels.cx, sizPixels.cy);
                if (hbm != NULL) {
                    // do render
                    HGDIOBJ hbmOld = ::SelectObject(hMemDC, hbm);
                    RECT rc = {0, 0, sizPixels.cx, sizPixels.cy};
                    ::FillRect(hMemDC, &rc,
                        reinterpret_cast<HBRUSH>(::GetStockObject(WHITE_BRUSH)));
                    pdfium.FPDF_RenderPage(hMemDC, pdf_page,
                        0, 0, sizPixels.cx, sizPixels.cy, 0,
                        FPDF_ANNOT | FPDF_PRINTING
                    );
                    ::SelectObject(hMemDC, hbmOld);
                }
                ::DeleteDC(hMemDC);
                ::ReleaseDC(m_hWnd, hDC);

                // close the page
                pdfium.FPDF_ClosePage(pdf_page);
            }
            // close the document
            pdfium.FPDF_CloseDocument(pdf_doc);
        }

        return hbm;
    }
    
    ...
    HBITMAP hbm = load_pdf_page("my_file.pdf", 0);
    ...

/////////////////////////////////////////////////////
// Katayama Hirofumi MZ (katahiromz) [ARMYANT]
// Homepage     http://katahiromz.web.fc2.com/eindex.html
// BBS          http://katahiromz.bbs.fc2.com/
// E-Mail       katayama.hirofumi.mz@gmail.com
/////////////////////////////////////////////////////
