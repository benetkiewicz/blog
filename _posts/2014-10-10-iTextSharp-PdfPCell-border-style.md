---
layout: post
title:  "iTextSharp PdfPCell border style"
date:   2014-10-10 16:57:59
categories: iTextSharp
---

Just a quick note about global styles in PDF documents created with iTextSharp, that may save someone a lot of time and experimenting.

I have a document which is divided into three vertical regions or partitions with a dashed line, like that:

{% highlight C# %}
public static void DrawDashedLines(Document doc, PdfWriter writer)
{
    PdfContentByte cb = writer.DirectContent;
    cb.SetLineDash(3f, 3f);
    cb.MoveTo(0, doc.PageSize.Height / 3);
    cb.LineTo(doc.PageSize.Width, doc.PageSize.Height / 3);
    cb.Stroke();
    // some code removed for clarity
}
{% endhighlight %}

Later, in each of the partitions, I have a table, where some of table's cells need to have solid border. All my cells had dashed border instead of solid:

{% highlight C# %}
var cell = new PdfPCell();
// not setting border to PdfPCell.NO_BORDER;
table.AddCell(cell);
{% endhighlight %}

 I wasted quite a lot of time before I realized that the state of `DirectContent` is preserved for a document and I set the dashed style in `DrawDashedLines` method. Resetting the style in the following manner fixed my issue and my table cells have solid borders again.

{% highlight C# %}
public static void ResetLineStyle(Document doc, PdfWriter writer)
{
    PdfContentByte cb = writer.DirectContent;
    cb.SetLineDash(0);
    cb.Stroke();
}
{% endhighlight %}
