# ISC 201 - Midterm

## Game Delivery
- [ ] This assignment will be graded as a Midterm, it should be built in `Release` mode and targeting `x64` systems.
  - [ ] If you add, or were adding external libraries, check your project settings so everything get's bundled properly when changing the build mode and the different architectures.
- [ ] The final build should be uploaded as a zipped file to this repository, only with the binary file and required dependencies for the game to run.

## Source Code Delivery
- [ ] The source code needs to be referenced as a module, accessible from this repository.
- [ ] If you haven't already please create a release for version **v0.0.5**. It should represent your work for *Assignment 5*.
- [ ] This assignment should be final, therefore it should be available as a tagged release as well. Use **v0.0.6**.

## New features

### Font rendering & Score
- [ ] Players should score points for destroying asteroids. You should come up with your own criteria for the score value per destroyed asteroid.
- [ ] There are no levels here, so you should incrementally add asteroids as the game progresses. The player should never run out of things to shoot at.
- [ ] Players should have 3 lives, and earn an extra life every `X` amount of points. Lives should be represented as ships and should be drawn on the top right corner of the screen. Take into account rendering adjustments for lives added or removed.
- [ ] Score should be rendered in the top right corner. Use SDL_ttf 2.0 for this. [Download](https://www.libsdl.org/projects/SDL_ttf/) the  development libraries for VC++ and reference them in your project. Remember to do this for all of the platforms and for all of the build modes. Please refer to the official documentation in order to learn how to use the library. In addition, there is a post build step that copies the 3rd party libraries DLLs to your build directory, you will need to add these three commands for the SDL_ttf library (you might need to do this for the sound and font files):

      copy /Y "$(SolutionDir)\Externals\SDL2_ttf-2.0.14\lib\x64\SDL2_ttf.dll" "$(TargetDir)SDL2_ttf.dll"
      copy /Y "$(SolutionDir)\Externals\SDL2_ttf-2.0.14\lib\x64\libfreetype-6.dll" "$(TargetDir)libfreetype-6.dll"
      copy /Y "$(SolutionDir)\Externals\SDL2_ttf-2.0.14\lib\x64\zlib1.dll" "$(TargetDir)zlib1.dll"

- [ ] Per the documentation you can initialize the library like this:
      
        if (TTF_Init() == -1) {
			  SDL_Log("TTF_Init: %s\n", TTF_GetError());
			  return false;
        }
- [ ] Per the documentation this is how you execute the library cleanup function:

       TTF_Quit();
       
- [ ] There are some debugging routines to make sure the library was loaded, you can have them in your `Init` function as well.

        SDL_version compile_version;
        const SDL_version *link_version = TTF_Linked_Version();
		SDL_TTF_VERSION(&compile_version);
    
		SDL_Log("compiled with SDL_ttf version: %d.%d.%d\n",
			compile_version.major,
			compile_version.minor,
			compile_version.patch);
      
		SDL_Log("running with SDL_ttf version: %d.%d.%d\n",
			link_version->major,
			link_version->minor,
			link_version->patch);
      
- [ ] You need to use `TTF_OpenFont` in order to load a font file. If the function fails to load the font it will return a `nullptr`.
  - [ ] Fonts need to be loaded once and only once.
- [ ] Fonts are first rendered to a texture and then to the screen. Since we have not learned how to do this, here's how you achieve it.
    
      unsigned int power_two_floor(unsigned int val) {
		unsigned int power = 2, nextVal = power * 2;
		while ((nextVal *= 2) <= val)
			power *= 2;
		return power * 2;
      }
    
    
      void RenderText(std::string message, SDL_Color color, float x, float y, int size)
	  {		
		glLoadIdentity();
		glTranslatef(x, y, 0.f);

		SDL_Surface *surface;

		//Render font to a SDL_Surface
		if ((surface = TTF_RenderText_Blended(m_font, message.c_str(), color)) == nullptr) {
			TTF_CloseFont(m_font);
			std::cout << "TTF_RenderText error: " << std::endl;
			return;
		}

		GLuint texId;

		//Generate OpenGL texture
		glEnable(GL_TEXTURE_2D);
		glGenTextures(1, &texId);
		glBindTexture(GL_TEXTURE_2D, texId);

		//Avoid mipmap filtering
		glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_NEAREST);
		glTexParameterf(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST);

		//Find the first power of two for OpenGL image 
		int w = power_two_floor(surface->w) * 2;
		int h = power_two_floor(surface->h) * 2;

		//Create a surface to the correct size in RGB format, and copy the old image
		SDL_Surface * s = SDL_CreateRGBSurface(0, w, h, 32, 0x00ff0000, 0x0000ff00, 0x000000ff, 0xff000000);
		
		SDL_BlitSurface(surface, NULL, s, NULL);

		//Copy the created image into OpenGL format
		glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, w, h, 0, GL_BGRA, GL_UNSIGNED_BYTE, s->pixels);

		//Draw the OpenGL texture as a Quad
		glBegin(GL_QUADS); {
			glTexCoord2d(0, 1); glVertex3f(0, 0, 0);
			glTexCoord2d(1, 1); glVertex3f(0 + surface->w, 0, 0);
			glTexCoord2d(1, 0); glVertex3f(0 + surface->w, 0 + surface->h, 0);
			glTexCoord2d(0, 0); glVertex3f(0, 0 + surface->h, 0);
		} glEnd();
		glDisable(GL_TEXTURE_2D);

		//Cleanup
		SDL_FreeSurface(s);
		SDL_FreeSurface(surface);
		glDeleteTextures(1, &texId);
	  }

- [ ] You will need to enable `GL_BLEND` in order to support color blending. This should also be done only once in your code, place this when you initialize your GL code.

      /* 
		 * Enable blending
		 * You can read more about blending here:
		 * http://www.informit.com/articles/article.aspx?p=1616796&seqNum=5
		 */
		glEnable(GL_BLEND);
		glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
    
- [ ] You should use one of the provided fonts or all of them and allow for some config setting. Your choice.
- [ ] When the player looses a life in the game, it should respawn on the center of the screen and have a cooling period of 2 seconds were he cannot die nor shoot asteroids.
- [ ] If the player depletes all of their lives a Game Over text should appear on top of the screen.
  - [ ] The player should be able to restart the game by pressing a specified key (via the game over screen).

### Sounds
[IrrKlang](http://www.ambiera.com/irrklang/downloads.html) is a high level 2D and 3D cross platform (Windows, Mac OS X, Linux) sound engine and audio library that plays WAV, MP3, OGG, and FLAC files to name a few. It also features several audio effects like reverb, delay and distortion that can be extensively tweaked. Please download the library files for the `x64` platform. Please read the documentation in order to learn how to use the library and familiarize with its API.

Please follow the same steps described above in order to add external dependencies. You need to add header references and linker references to the project. You will also need to do a post build step that includes this dependency for any build.

Once you included the header files of irrKlang, added their library (irrKlang.lib) to the linker settings and copied their dll files to the appropriate locations (usually the same location where the .exe resides) we're set to go. Note that if you want to load MP3 files you'll also have to include the ikpMP3.dll file.

This is a small sammple snippet that demonstrates how to use the library

       #include <irrklang/irrKlang.h>

       ISoundEngine *SoundEngine = irrklang::createIrrKlangDevice();
  
        void Game::Init()
        {
           [...]
           SoundEngine->play2D("audio/thrust.wav", GL_TRUE);
        }
	
Please use the sounds provided in this repo or feel free to produce your own. It is expected to have sounds for: shooting, thrust, game over, extra ships, destroying big, medium and small asteroids.

## Grading
Since this is a midterm evaluation will be different. The overall grade will be 20% of the total acummulated points for the class. The distribution of this grade will be as follows:

* 50% of the grade will be attributed using the grading Rubric attached to this repository.
* 50% of the grade will be attributed using the following "GAMING EXPERIENCE" criteria:
  * 20% End-user game experience. This defines how good the game feels or how close is the experience to the real game.
  * 30% Text rendering
  * 30% Sound support
  * 10% Debug mode
  * 10% Respawing, game over and re-starting the game
	
## Extra credit
Adding the **flying saucer** adds an extra 5% to your overall grade. These are the requirements:

- [ ] The flying saucer must appear randomly while playing the game.
- [ ] The flying saucer should fly across the screen (end-to-end) and then disappear.
- [ ] The flying saucer should shoot the user randomly, bullets should follow a constant trajectory and disappear after 3 or 4 seconds.
- [ ] As the player has a higher score the amount of bullets and the saucer speed should increase.

Attribution of the extra credit will only happen if all of the "GAMING EXPERIENCE" requirements are met. 
