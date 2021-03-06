======================================
Placing clips according to a picture
======================================

So how do you do some complex compositing like this ?

.. raw:: html

        <center>
        <object><param name="movie"
        value="http://www.youtube.com/v/1hdgNxX-tas&hl=en_US&fs=1&rel=0">
        </param><param name="allowFullScreen" value="true"></param><param
        name="allowscriptaccess" value="always"></param><embed
        src="http://www.youtube.com/v/1hdgNxX-tas&hl=en_US&fs=1&rel=0"
        type="application/x-shockwave-flash" allowscriptaccess="always"
        allowfullscreen="true" width="550" height="450"></embed></object>
        </center>

It takes a lot of bad taste and... the right tools !

So we first make a picture that indicates the regions where the clips will be. This picture (that I made with Inkscape) is a *.png* that looks like this:

.. figure:: motif.jpeg

The next function finds the regions in the picture. For each region found it returns a mask that defines the region, and a tuple ``(x1,y1,x2,y2)`` defining a bounding rectangle giving the position of the region in the picture. ::
    
    import scipy.ndimage as spim
    from pylab import *

    def pic2regions(pic,render=True, crop=True):
        """
        Finds the different regions contained in a  delimited by
        picture as a list of elements (maskClip,(x1,y1,x2,y2)).
        maskClip is a clip ready to be used as a mask delimiting
        the region. The resolution of maskClip is that of 'pic'
        if crop=True, else, it is the size of the region.
        render=True pops a picture rendering which region has
        which number    
        """
        
        if len(pic.shape)>2:
            pic = pic[:,:,0]
        objects, num = spim.label(pic)
        num = num+1
        masks = [(objects==i).astype(float) for i in range(num)]
        mamasks = [np.ma.masked_values(m,0) for m in masks]
        h,w = pic.shape
        XX,YY = [a for a in np.meshgrid(range(w),range(h))]
        XXc = [ XX*c for c in mamasks]
        YYc = [ YY*c for c in mamasks]
        boxes = [ map(int,(xxc.min(),yyc.min(),xxc.max(),yyc.max()))
                  for xxc, yyc in zip(XXc, YYc)]
        
        if crop:
            masks = [ m[y1:y2,x1:x2] 
                      for m,(x1,y1,x2,y2) in zip (masks,boxes)] 
        print [m.shape for m in masks]
        
        maskClips = [ImageClip(m) for m in masks]
        
        if render:
            print "found %d objects"%(num)
            fig,ax = subplots(2)
            ax[0].axis('off')
            ax[0].imshow(objects)
            ax[1].imshow([range(num)],interpolation='nearest')
            ax[1].set_yticks([])
            
        return zip(maskClips,boxes)
        
This function returns a list of the regions found. To help visualize which element of the list corresponds to which region in the picture, ``pic2regions`` splashes a Matplotlib figure giving the index of each region. For our picture, it returned this: ::
    
    im = imread("./ultracompositing/motif.png")    
    regs = pic2regions(im)

.. figure:: matplotlibCompo.jpeg

Finally, we load 8 different video clips that we assign to the different regions. In our video, this implied for each of these clips

- Resizing the clip so that it will fill the region.
- Placing each clip at its position on the picture
- Giving to each clip a mask that will shape it like the region.

So here is the final script: ::
    
    # We load height clips from the US National Parks.
    # These are public domain. :D
    clips = [MovieClip(n).subclip(20,25) for n in
         [ "./videos/romo_0004.mov",
          "./videos/apis-0001.mov",
          "./videos/romo_0001.mov",
          "./videos/elma_s0003.mov",
          "./videos/elma_s0002.mov",
          "./videos/calo-0007.mov",
          "./videos/grsm_0005.mov",
          "./videos/grsm_0002.mov"]]

    im = imread("./ultracompositing/motif.png")
    size = w,h = im.shape[:2][::-1]
    regs = pic2regions(im,crop=True)
    # eliminate the first region found (it is the border)
    regs = regs[1:]
    masks = [r[0] for r in regs] 
    poss = [(x1,y1) for m,(x1,y1,x2,y2) in regs]
    comp_clips =  [c.resize(m.size).set_mask(m).set_pos(p)
                   for c,m,p in zip(clips,masks,poss)]

    cc = CompositeVideoClip(size,comp_clips)
    cc.resize(0.5).preview()

Nice and short ! And you can get very creative with this kind of tricks.

