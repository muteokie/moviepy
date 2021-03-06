=============================================
Freezing a movie frame with a painting effect
=============================================

That's an effect that we have seen a lot in westerns and such.

.. raw:: html

        <center>
        <object><param name="movie"
        value="http://www.youtube.com/v/aC5CifkacSI&hl=en_US&fs=1&rel=0">
        </param><param name="allowFullScreen" value="true"></param><param
        name="allowscriptaccess" value="always"></param><embed
        src="http://www.youtube.com/v/aC5CifkacSI&hl=en_US&fs=1&rel=0"
        type="application/x-shockwave-flash" allowscriptaccess="always"
        allowfullscreen="true" width="550" height="450"></embed></object>
        </center>

Before
starting let us see how to make a photo look more like a painting. It
can be done easy as follows:

- Find the edges of the image with the Sobel algorithm. What you obtain
  is what looks like a hand-drawing of the photo.
- Multiply the image to make the colors flashier, and add the contours
  obtained at the previous step. 

Which is implemented in just a few lines of code: ::
    
    from moviepy import *
    from skimage.filter import sobel

    def to_painting(image,saturation = 1.4,black = 0.006):
        """ transforms any photo into some kind of painting """
        edges = sobel(image.mean(axis=2))
        darkening =  black*(255*np.dstack(3*[edges]))
        painting = saturation*image-darkening
        return np.maximum(0,np.minimum(255,painting)).astype('uint8')
        
Now let us see the actual script. We work on an Audrey Hepburn clip taken
from *Charade*, a movie which is not copyrighted due to a copyright
omission (good for us).

The final clip will be the concatenation of three part: the part before
the effect, the part with the effect, and the part after the effect.
The part with the effect is obtained as follows:

- Take the frame to freeze and make a "painted image" of it. Make it a clip.
- Add a text clip saying "Audrey" to the "painted image" clip.
- Overlay the painted clip over the original frame, but make it appear and
  disappear with a fading effect.

Here you are for the code: ::
    
    charade = MovieClip("./videos/charade.mp4", audio=False)
    tf = 19*60+21.0 # Time of the freeze, 19'21

    # WE TAKE THE SUBCLIPS WHICH ARE 2 SECONDS BEFORE & AFTER THE FREEZE
    
    clip_before = charade.subclip(tf -2,tf )
    clip_after = charade.subclip(tf ,tf +2)

    
    # THE FRAME TO FREEZE
    
    im_freeze = charade.get_frame(tf)
    painting = ImageClip(to_painting(im_freeze))
    txt = TextClip('Audrey',font='Amiri-regular',fontsize=35)
    painting_txt = CompositeVideoClip(charade.size,
                    [painting,txt.set_pos((10,180))])

    # FADEIN/FADEOUT EFFECT ON THE PAINTED IMAGE
    
    painting_fading = CompositeVideoClip(charade.size,
                        [ImageClip(im_freeze),
                         painting_txt.with_mask().fadein(0.3).
                                             set_duration(3).
                                                fadeout(0.3)])
    
    # FINAL CLIP AND RENDERING
    
    final_clip =  concat([ clip_before,
                           painting_fading.set_duration(3.6),
                           clip_after])
    
    final_clip.to_movie('audrey.avi',fps=charade.fps)
