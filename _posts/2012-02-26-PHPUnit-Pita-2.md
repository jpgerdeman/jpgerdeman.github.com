---
layout: post
title: "PHPUnit PITA 2"
---
Continuing from my last post.

Well I did have the need to run two versions of phpunit. I could've just created another VM for that and install everything into the VM, but aside from needing two versions of phpunit there was no real reason to do so. On the contrary I needed both for the same project. Another reason virtual images were no solution was that my coworkers also needed phpunit and did not want to go through the hassle of installing it themselves or runnig yet another virtual image.

To make a long story short, I created a standalone phar file for all of us.

I downloaded PHPUnit in version 3.5.15 unpacked it to a dummy PEAR Folder and had a look at the package.xml. This file lists all the dependencies of a PEAR Package including the version of the package and its pear channel.

Most packages were from the PHPUnit channel anyways, so downloading the required packages was no pain. I unpacked all the files into my dummy PEAR folder. Make sure to look at the package.xml for the yaml package as the file structure of the zip file differs from the file structure it should have in the PEAR folder.

I then deleted all batch files and php files from the main folder, excluding the phpunit ones. I removed the phpunit batch file from the folder, but did not delete it. 


	class PHPUnitPackager
    {
		private $pharname = null;
		private $packageLocation = null;
		private $outputPath = null;
		private $stub = null;
		public function run()
		{
			$this->resetPhar();

			$this->log('creating new phar Archive '. $this->getPharName().' for '.$this->getPackageLocation());
			$phar = new Phar($this->getPharname(), 0, $this->getPharname());
			$phar->compressFiles(Phar::GZ);
			$phar->setSignatureAlgorithm (Phar::SHA1);
			$phar->startBuffering();
			$phar->buildFromDirectory($this->getPackageLocation());
			$phar->stopBuffering();
			if( is_null($this->getStub()) )
			{
				$stub = $phar->createDefaultStub('stub.php');
			}
			else
			{
				$stub = $this->getStub();
			}
			$phar->setStub($stub);
			$phar = null;
		}

		private function resetPhar()
		{
			@unlink($this->getPharname());
		}

		private function log( $msg )
		{
			$time = date( 'H:i:s' );           
			echo $time." ".$msg.PHP_EOL;
		}

		public function getPharname()
		{
			return $this->pharname;
		}

		public function setPharname( $pharname )
		{
			$this->pharname = $pharname;
			return $this;
		}

		public function getPackageLocation()
		{
			return $this->packageLocation;
		}

		public function setPackageLocation( $packageLocation )
		{
			$this->packageLocation = $packageLocation;
			return $this;
		}

		public function getOutputPath()
		{
			return $this->outputPath;
		}

		public function setOutputPath( $outputPath )
		{
			$this->outputPath = $outputPath;
			return $this;
		}
		public function getStub()
		{
			return $this->stub;
		}

		public function setStub( $stub )
		{
			$this->stub = $stub;
			return $this;
		}

	}

    $stub = '
		<?php
			// make sure that the phar file is checked first for files
			set_include_path('phar://phpunit_3_5_15.phar'.PATH_SEPARATOR.get_include_path());
			include "phar://phpunit_3_5_15.phar/phpunit.php";
			__HALT_COMPILER();
    ';
    $packager = new PHPUnitPackager();       
    $packager
		->setPackageLocation(__DIR__.'/phpunit_phar/')
		->setOutputPath(__DIR__)
		->setPharName('phpunit_3_5_15.phar')
		->setStub( $stub )
		->run();
 
That was nice and easy. PHPUnit can now be invoked by 

    php phpunit_3_5_15.phar

If you're on windows you wil also need a batch file. The batch file needs to be altered to do the above invocation and looks like this 
view sourceprint?

    @echo off
    if "%PHPBIN%" == "" set PHPBIN=C:\path\to\php.exe
    if not exist "%PHPBIN%" if "%PHP_PEAR_PHP_BIN%" neq "" goto USE_PEAR_PATH
    GOTO RUN
    :USE_PEAR_PATH
    set PHPBIN=%PHP_PEAR_PHP_BIN%
    :RUN
    "%PHPBIN%" "phpunit_3_5_15.phar" %*


That's it. Now why doesn't PEAR offer something like that?