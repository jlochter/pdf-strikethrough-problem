# pdf-strikethrough-problem

When extracting pieces of text from a PDF, often libraries do not represent and capture visual representations like strikethrough or underline, although text is retrieved.
It means that after the text is extracted no metadata is generated with that information.

As an example, I will provide this Alabama bill: https://alison.legislature.state.al.us/files/pdfdocs/SearchableInstruments/2024RS/HB465-int.pdf

![image](https://github.com/jlochter/pdf-strikethrough-problem/assets/5498030/c630e644-8d88-4fa8-a499-531d06615b1a)

The main issue is the representation of those lines with geometric objects, instead text tags or metadata, as we would see in other markup languages (HTML or Markdown).
Thus to identify which text is underlined or strikedthrough is necessary to compare geometric position of lines objects and texts objects.

Luckly, both information is avaliable and can be used. I can show it with the following snippet:

```python
from pdfminer.high_level import extract_pages
from pdfminer.layout import LTTextContainer, LTChar

for page_layout in [list(extract_pages("HB465-int.pdf"))[2]]:
    for element in page_layout:
        print(element)
```

In the output, you will find those lines, those are interesting for our logical line:

```
<LTTextBoxHorizontal(37) 122.000,536.400,504.200,548.400 '(1) Any person individual adjudicated by a court of\n'>
<LTRect 174.500,542.350,227.000,543.100>
<LTRect 227.000,536.450,309.500,537.200>
```

As you can see, in the LTTextBox (text object in PDF idiom) both **person** and **individual** were captured.
Another thing, there are two LTRect (line/rect in PDF idiom) in those coordinates. One for strikethrough, another for underline.
But which one is the strikethrough? You can compare coordinates from LTText to LTRect. They are boundary box coordinates, which is usually represented with x=0, y=0 in the left-bottom corner of page.
If you see coordinates for LTText (122.000,536.400,504.200,548.400) as x0,y0,x1,y1 you will understand that this line vertically is between 536 and 548.
So, whatever LTRect is closer to 536 that one is the underline, and whatever is closer to middle would be the strikethrough.

```
Strikethrough: <LTRect 174.500,542.350,227.000,543.100>
Underline: <LTRect 227.000,536.450,309.500,537.200>
```

Ok, and what about what part of text is strikedthrough or underlined? Then we need to compare x0 and x1, and you can see from lines above that one ends another starts, like this:

![image](https://github.com/jlochter/pdf-strikethrough-problem/assets/5498030/0f9c5a21-ea59-443a-9593-ff92dc4a851c)

Now comes the dirty-ugly work, because we need to get every character from that LTText and check if it is in the LTRect range.

So, first to see how get each character, we can use the following snippet:

```python
from pdfminer.high_level import extract_pages
from pdfminer.layout import LTTextContainer, LTChar

for page_layout in [list(extract_pages("HB465-int.pdf"))[2]]:
    print('--------------------------')
    for element in page_layout:
        if isinstance(element, LTTextContainer):
            for text_line in element:
                if 'Any person individual adjudicated by a court' in str(text_line): # filtering to not be so verbose
                  print(text_line)
                  for character in text_line:
                    print('\t',character)
```

Output:

```
<LTTextLineHorizontal 122.000,536.400,504.200,548.400 '(1) Any person individual adjudicated by a court of\n'>
	 <LTChar 122.000,536.400,129.200,548.400 matrix=[1.00,0.00,0.00,1.00, (122.00,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='('>
	 <LTChar 129.500,536.400,136.700,548.400 matrix=[1.00,0.00,0.00,1.00, (129.50,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='1'>
	 <LTChar 137.000,536.400,144.200,548.400 matrix=[1.00,0.00,0.00,1.00, (137.00,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text=')'>
	 <LTChar 144.500,536.400,151.700,548.400 matrix=[1.00,0.00,0.00,1.00, (144.50,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text=' '>
	 <LTChar 152.000,536.400,159.200,548.400 matrix=[1.00,0.00,0.00,1.00, (152.00,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='A'>
	 <LTChar 159.500,536.400,166.700,548.400 matrix=[1.00,0.00,0.00,1.00, (159.50,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='n'>
	 <LTChar 167.000,536.400,174.200,548.400 matrix=[1.00,0.00,0.00,1.00, (167.00,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='y'>
	 <LTChar 174.500,536.400,181.700,548.400 matrix=[1.00,0.00,0.00,1.00, (174.50,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text=' '>
	 <LTChar 182.000,536.400,189.200,548.400 matrix=[1.00,0.00,0.00,1.00, (182.00,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='p'>
	 <LTChar 189.500,536.400,196.700,548.400 matrix=[1.00,0.00,0.00,1.00, (189.50,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='e'>
	 <LTChar 197.000,536.400,204.200,548.400 matrix=[1.00,0.00,0.00,1.00, (197.00,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='r'>
	 <LTChar 204.500,536.400,211.700,548.400 matrix=[1.00,0.00,0.00,1.00, (204.50,540.00)] font='OQCNNE+CourierNewPSMT' adv=7.199999999999999 text='s'>
```

Now we need to collect geometric objects and compare if we have one in the middle of vertical range of a given row. In that case, we don't retrieve that caracter.
So we create a function to check overlap and remove those characters that overlap with a strikethrough, but keep those underlines:

```python
from pdfminer.high_level import extract_pages
from pdfminer.layout import LTTextContainer, LTChar, LTRect

def intersects(c, overlap_lines):
    for line in overlap_lines:
      c_x0,_,c_x1,_ = c.bbox
      line_x0,_,line_x1,_ = line.bbox
      if c_x0 >= line_x0 and c_x1 <= line_x1:
        return True
    return False
        

for page_layout in [list(extract_pages("HB465-int.pdf"))[2]]:
    lines = []
    for element in page_layout:
      if isinstance(element, LTRect):
          lines.append(element)
    for element in page_layout:
      overlap_lines = []
      if isinstance(element, LTTextContainer):
        # check which lines overlap with text element
        _,text_y0,_,text_y1 = element.bbox
        middle = (text_y0 + text_y1)/2
        for line in lines:
          _,line_y0,_,line_y1 = line.bbox
          if (text_y0 <= line_y0 <= text_y1) and (middle-1 <= line_y1 <= middle+1):
            overlap_lines.append(line)
        # process characters overlaps with strikethrough
        for text_line in element:
          line = [c._text for c in text_line if isinstance(c, LTChar) and not intersects(c, overlap_lines)]
          line = ''.join(line)
          print(line)
```

Output example (notice we don't have **person** in (1) and (3):

```
(1) Any individual adjudicated by a court of
this state to be the father of a child born out of wedlock.
(2) Any individual who has filed a notice of
intent to claim paternity of the child with the registry
before or after the birth of a child born out of wedlock, 
which
includes the information required in subsection (c).
(3) Any individual adjudicated by a court of
another state or territory of the United States to be the
father of a child born out of wedlock, where a certified copy
of the court order has been filed with the registry by the
 individual or any other individual.
```
