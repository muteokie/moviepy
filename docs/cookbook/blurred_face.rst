========================================
Tracking and blurring someone's face
========================================



.. raw:: html

        <center>
        <object><param name="movie"
        value="http://www.youtube.com/v/FWCKYTRCrBI&hl=en_US&fs=1&rel=0">
        </param><param name="allowFullScreen" value="true"></param><param
        name="allowscriptaccess" value="always"></param><embed
        src="http://www.youtube.com/v/FWCKYTRCrBI&hl=en_US&fs=1&rel=0"
        type="application/x-shockwave-flash" allowscriptaccess="always"
        allowfullscreen="true" width="550" height="450"></embed></object>
        </center>

First we will need to track the face, i.e. to get two functions ``fx`` and ``fy`` such that ``(fx(t),fy(t))`` gives the position of the center of the head at time ``t``. This will be easily done with
:ref:`VideoClip.manual_tracking`. Then we will need to blur the area of the video around the center of the head.

To track an object in a video you write: ::
    
    txy = myClip.manual_tracking(0,5,fps=5)

This will display the frames of ``myClip`` between t=0s and t=5s and on each frame it will wait for you to click on the face. It then returns a list ``[(t1,x1,y1),(t2,x2,y2)]`` of the positions that you have clicked.

The next snippet uses these data to make the two functions ``(fx(t),fy(t))`` discussed above: ::
    
    from scipy.interpolate import interp1d

    def txy2functions(txyList):
        """
        Takes the output of VideoClip.manual_tracking and returns
        2 functions (fx,fy) where fx : t -> x(t) and fy : t -> y(t)
        """
        tt,xx,yy = zip(*txyList)
        fx = interp1d(tt,xx,bounds_error=False, fill_value=xx[0])
        fy = interp1d(tt,yy,bounds_error=False, fill_value=yy[0])
        return fx,fy

And the coming function makes a filter that blurs the regions around ``(fx(t),fy(t))``: ::
    
    import cv2
    
    def blur_filter(fx,fy,r_zone,r_blur=None):
        """
        Returns a filter that will blurr a moving part of
        the frames. The position of the blur at time t is
        defined by (fx(t), fy(t)), the radius of the blurring
        by r_zone and the intensity of the blurring by r_blur.
        """
        
        if r_blur==None: r_blur = 2*r_zone/3
        
        def fl(gf,t):
            im = gf(t)
            h,w,d = im.shape
            x,y = int(fx(t)),int(fy(t))
            x1,x2 = max(0,x-r_zone),min(x+r_zone,w)
            y1,y2 = max(0,y-r_zone),min(y+r_zone,h)
            reg_res = y2-y1,x2-x1
            
            # We will now make a circled mask. 
            # IN older OpenCV, instead of the next two lines, write
            # mask = cv2.circle(np.zeros(reg_res).astype('uint8'),
            #   (r_zone,r_zone),r_zone,255,-1, lineType=cv2.LINE_AA)
            mask = np.zeros(reg_res).astype('uint8')
            cv2.circle(mask, (r_zone,r_zone),r_zone,255,-1,
                                   lineType=cv2.CV_AA)
            #-----
            
            mask = np.dstack(3*[(1.0/255)*mask])
            
            orig = im[y1:y2,x1:x2]
            blurred = cv2.blur(orig,(r_blur,r_blur))
            im[y1:y2,x1:x2] = mask*blurred + (1-mask)*orig
            return im
        
        return fl

Finally, here is the script: ::
    
    from moviepy import *
    import pickle
    
    # Because we will work with sound and we do not want to carry around
    # the sound of the whole original movie, we first extract the
    # interesting part of the movie (run this line only once): 

    ffmpeg.extract_subclip("./videos/chaplin.mp4",411.7,421.7+3,
                           "charlotSub.mp4")

    # Now we have a shorter movie that we can load
    chaplin = MovieClip("charlotSub.mp4", audio=True)
    clip = chaplin.subclip(0, chaplin.duration-3)


    # THE THREE NEXT LINES ARE FOR THE MANUAL TRACKING AND
    # MUST BE COMMENTED ONCE THE TRACKING HAS BEEN DONE
    # (AT FIRST RUN OF THE SCRIPT)
    txyList = clip.manual_tracking(fps=6)
    with open("chaplin_txy.dat",'w+') as f:
        pickle.dump(txyList,f)

    # Load the tracking data (see comment above)
    with open("chaplin_txy.dat",'r') as f:
        fx,fy = txy2functions(pickle.load(f))


    # blur the head in the clip
    clip_blurred = clip.fl(blur_filter(fx,fy,25))

    # Generate the text
    txt = TextClip("Hey you ! \n You're blurry!", color='grey',
                   font = "Century-Schoolbook-Italic", fontsize=40)

    # Put the text on a grey background
    txt = txt.on_color(chaplin.size,(25,25,25),("center","center"))

    # Concatenate the clip with the blur and the clip with text
    # Set the audio of concat so that there will also be music
    # when the text is playing.
    
    final = concat([clip_blurred,txt.set_duration(3)])
    final = final.set_audio(chaplin.audio)

    # write to a file. Here we use the 'XVID' codec because
    # 'raw' is too heavy and 'DIVX' is too ugly.
    
    final.to_movie('blurredChaplin.avi',fps=24, codec='XVID')
    
    # This 13s movie with sound and special effects was generated
    # in six seconds.
