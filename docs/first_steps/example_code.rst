Example code
------------

We start with some code to show you the MoviePy syntax. In a typical script, you load or generate some clips, modify them, combine them, and write the result to a video file.

We will now load a movie of my last holidays, lower the volume, add a title in the center of the video for the ten first seconds, and write the result in a file: ::
    
    from moviepy import *
    
    # Load myHolidays.mp4 and select the subclip 00:00:50 - 00:00:60
    clip = MovieClip("myHolidays.mp4", audio=True).subclip(50,60)

    # Reduce the sound volume (volume x 0.8)
    clip.sound = clip.sound.volumex(0.8) 
    
    # Generate a text clip. You can customize the font, color, etc.
    txt_clip = TextClip("My Holidays 2013",fontsize=70,color='white')
    
    
    # Say that you want it to appear 10s at the center of the screen
    txt_clip = txt_clip.set_pos(('center','center')).set_duration(10)
    
    # Overlay the text clip on the first video clip
    video = CompositeVideoClip(clip.size, [clip, txt_clip])
    
    # Write the result to a file. The default encoding will be DIVX.
    video.write("myHolidays_edited.avi",fps=24) 

See ? That was easy !

