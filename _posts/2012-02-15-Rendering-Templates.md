---
layout: post
title: "Rendering Templates to Files in PHP"
---

So I recently had to generate a bunch of files with contents from a database. Since we're already using Symfony2 and Doctrine I opted for implementing this in PHP as a Symfony Command, instead of using another language.

I wrote PHP templates for the files and called them from my Command like this and wrote the return value to a file.

	function renderTemplate( $template, $parameter )
	{
	 ob_start();
	 extract($parameter);
	 include $template;
	 return ob_get_clean();
	}

Files started to get big real fast and I soon encountered the following:

	zend__mm_buffer corruption

Well I could've changed my buffer size in the php.ini locally, but our admin would've never allowed it on the production servers. So that was no option.

I checked the manual to see if ob_start allowed for a callback and indeed it did. Sadly the callback is only called on every flush of the cache. The only way I know of flushing the cache are the ob_get_clean and similar functions, which activate after everything is done :(. So that wouldn't work.

The manual mentioned more parameters for the ob_start, so I checked them. It took me a bit of looking around and I reread the manual a few times before I finally found the text explaining the chunk size. They couldnt've hidden the darn thing any better.

Anyway my code then looked like this

	function renderTemplateToFile( $template, $parameter )
	{
		ob_start('mycallback',2);
		extract($parameter);
		include $template;
		ob_get_clean();
	}

	function mycallback($text)
	{
		file_put_contents('file.out', $text, FILE_APPEND);
	}

This didn't work. It took me a bit of figuring out, but there seem to be two problems here.
 - The Callback should always return a string
 - <del>if you don't asign a variable to the output of ob_get_clean the callback won't get fired. At least not on all the time on all systems.</del> Seems I had a typo in one of my statements, which caused this unusual behavior

The result of this was the following class, which of course can still be further improved:

	class TemplateToFileRenderer
	{
		protected $_params = array();
		protected $_filename = null;
		public function setParameter( $params )
		{
			$this->_params = $params;
			return $this;
		}
		public function setFilename( $filename )
		{
			$this->_filename = $filename;
			return $this;
		}
		public function render( $template )
		{
			$this->resetFile();
			ob_start( array($this, 'writeBuffer'), 2 );
			extract($this->_params);
			include_once($template);
			ob_get_clean();
		}
		private function resetFile()
		{
			@unlink($this->_filename);
			touch($this->_filename);
		}
		public function writeBuffer( $text )
		{
			file_put_contents($this->_filename, $text, FILE_APPEND);
			return '';
		}
	}