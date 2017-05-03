# Adobe-Connect-Recording-Downloader
A collection of scripts that can be set up to automatically download MP4 files of Adobe Connect recordings.

This program finds new recordings in an Adobe Connect instance and automatically downloads the MP4 recordings. This program runs Adobe Connect and uses the Sikuli screenreader to interpret the recording and click buttons and enter info into input fields. Therefore it needs to be set up on a server with graphic user interface. I set it up on a Windows Server 2012 R2, with the following software
•	Python 2.7
•	Sikuli IDE 1.0.0 Win32/Win64
•	PHP 5.3 with PEAR extension
•	MYSQL
The PHP programs populate a MYSQL database. A Windows Batch file calls the Sikuli scripts that launch up to five recordings simultaneously in Adobe Connect, and then stores the recordings in folders named after the Adobe Connect meeting rooms. After it finishes downloading all the pending recordings in the hopper, it launches the PHP program to check for new recordings. It continues to check for new recordings every hour or two.

For PHP programs the institution-dependent constants are in the file int_config.php

For Python (Sikuli) programs the institution-dependent constants are in the file rec_config.py

Before attempting to run the programs, you should set up a MYSQL database with the following tables:

-- PHP Version: 5.3.26
-- --------------------------------------------------------
--
-- Table structure for table `archive_views`
--

CREATE TABLE IF NOT EXISTS `archive_views` (
  `starttime` varchar(25) DEFAULT NULL,
  `start_raw` int(14) NOT NULL,
  `login` varchar(30) DEFAULT NULL,
  `course` varchar(25) DEFAULT NULL,
  `recname` varchar(90) DEFAULT NULL,
  `duration` int(11) DEFAULT NULL,
  `useragent` varchar(160) DEFAULT NULL,
  `ip` varchar(20) DEFAULT NULL,
  `basestring` text,
  `endtime` varchar(25) DEFAULT NULL,
  `total` int(10) DEFAULT NULL,
  `comment` text,
  `end_raw` int(14) NOT NULL,
  `session` char(50) NOT NULL,
  `rec_url` varchar(100) NOT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

-- --------------------------------------------------------

--
-- Table structure for table `cc_configdetails`
--

CREATE TABLE IF NOT EXISTS `cc_configdetails` (
  `root_path` varchar(255) DEFAULT NULL,
  `search_criteria` varchar(255) DEFAULT NULL,
  `number_of_instances` int(11) DEFAULT NULL,
  `download_from_date` date DEFAULT NULL,
  `status_check` datetime DEFAULT NULL,
  `Stupid_unique` tinyint(1) DEFAULT NULL,
  UNIQUE KEY `root_path` (`root_path`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `cc_download_log`
--

CREATE TABLE IF NOT EXISTS `cc_download_log` (
  `scoid` int(11) NOT NULL COMMENT 'scoid of recording',
  `date` date DEFAULT NULL,
  `start_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `end_time` timestamp NOT NULL DEFAULT '0000-00-00 00:00:00',
  `num_downloaded` int(11) DEFAULT NULL,
  PRIMARY KEY (`scoid`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `cc_recordings_archive`
--

CREATE TABLE IF NOT EXISTS `cc_recordings_archive` (
  `scoid` varchar(255) DEFAULT NULL,
  `foldername` varchar(255) DEFAULT NULL,
  `instructor` tinytext COMMENT 'folder name should be instructor',
  `url` varchar(255) DEFAULT NULL,
  `meetingname` varchar(255) DEFAULT NULL,
  `meetingurl` varchar(255) DEFAULT NULL,
  `recordingname` varchar(255) DEFAULT NULL,
  `datecreated` varchar(255) DEFAULT NULL,
  `duration` int(32) DEFAULT NULL,
  `attendance` smallint(4) DEFAULT NULL COMMENT 'from Adobe API',
  `date_modified` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  UNIQUE KEY `scoid` (`scoid`),
  UNIQUE KEY `scoid_2` (`scoid`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `cc_recordings_in_ac`
--

CREATE TABLE IF NOT EXISTS `cc_recordings_in_ac` (
  `scoid` varchar(255) DEFAULT NULL,
  `foldername` varchar(255) DEFAULT NULL,
  `instructor` tinytext COMMENT 'folder name',
  `url` varchar(255) DEFAULT NULL,
  `meetingname` varchar(255) DEFAULT NULL,
  `meetingurl` varchar(255) DEFAULT NULL,
  `recordingname` varchar(255) DEFAULT NULL,
  `datecreated` varchar(255) DEFAULT NULL,
  `duration` int(32) DEFAULT NULL,
  `attendance` smallint(4) DEFAULT NULL COMMENT 'student attendance',
  `date_modified` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  UNIQUE KEY `scoid` (`scoid`),
  UNIQUE KEY `scoid_2` (`scoid`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `cc_recordings_saved`
--

CREATE TABLE IF NOT EXISTS `cc_recordings_saved` (
  `scoid` varchar(255) DEFAULT NULL,
  `status` char(10) DEFAULT 'P',
  `datedownloaded` datetime DEFAULT NULL,
  UNIQUE KEY `scoid` (`scoid`)
) ENGINE=MyISAM DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `cc_rec_attendance`
--

CREATE TABLE IF NOT EXISTS `cc_rec_attendance` (
  `scoid` int(11) DEFAULT NULL COMMENT 'recording scoid',
  `session_id` int(11) DEFAULT NULL COMMENT 'meeting session id#',
  `transcript_id` int(11) DEFAULT NULL COMMENT 'unique # for each login',
  `course_sco` int(11) DEFAULT NULL COMMENT 'sco for the course',
  `person` tinytext,
  `status` tinytext COMMENT 'type of enrollment',
  `start` datetime NOT NULL,
  `stop` datetime NOT NULL,
  `seconds` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `errors`
--

CREATE TABLE IF NOT EXISTS `errors` (
  `time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `useragent` varchar(120) DEFAULT NULL,
  `ip` varchar(20) DEFAULT NULL,
  `login` varchar(25) DEFAULT NULL,
  `course` varchar(45) DEFAULT NULL,
  `error` varchar(300) DEFAULT NULL,
  `number` int(11) NOT NULL AUTO_INCREMENT,
  PRIMARY KEY (`number`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 AUTO_INCREMENT=1 ;

-- --------------------------------------------------------

--
-- Table structure for table `live_courses`
--

CREATE TABLE IF NOT EXISTS `live_courses` (
  `course_code` tinytext NOT NULL,
  `name` tinytext NOT NULL COMMENT 'course name',
  `prof` tinytext NOT NULL,
  `prof_id` tinytext,
  `description` tinytext COMMENT 'Adobe Connect description',
  `connect_id` int(10) unsigned NOT NULL DEFAULT '0',
  `canvas_id` int(10) unsigned DEFAULT NULL,
  `department_canvas` smallint(6) DEFAULT '1' COMMENT 'Canvas department number',
  `enrollment` tinyint(4) DEFAULT NULL COMMENT 'from Canvas API',
  `start` date NOT NULL,
  `end` date NOT NULL,
  `comment` tinytext,
  `day` tinytext NOT NULL,
  `time` tinytext,
  `visible` tinyint(1) NOT NULL DEFAULT '1',
  `last_updated` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`connect_id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1;

-- --------------------------------------------------------

--
-- Table structure for table `recs`
--

CREATE TABLE IF NOT EXISTS `recs` (
  `course` varchar(25) DEFAULT NULL,
  `course_sco` int(12) DEFAULT NULL,
  `rec_name` varchar(72) DEFAULT NULL,
  `rec_url` varchar(43) DEFAULT NULL,
  `rec_sco` int(12) DEFAULT NULL,
  `date` char(30) DEFAULT NULL,
  `duration` int(12) DEFAULT NULL,
  `in_db` tinyint(1) DEFAULT NULL,
  `in_archive` tinyint(1) DEFAULT NULL,
  UNIQUE KEY `rec_sco` (`rec_sco`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

-- --------------------------------------------------------

--
-- Table structure for table `views`
--

CREATE TABLE IF NOT EXISTS `views` (
  `starttime` varchar(25) DEFAULT NULL,
  `start_raw` int(14) NOT NULL,
  `login` varchar(30) DEFAULT NULL,
  `course` varchar(25) DEFAULT NULL,
  `recname` varchar(90) DEFAULT NULL,
  `duration` int(11) DEFAULT NULL,
  `useragent` varchar(160) DEFAULT NULL,
  `ip` varchar(20) DEFAULT NULL,
  `basestring` text,
  `endtime` varchar(25) DEFAULT NULL,
  `total` int(10) DEFAULT NULL,
  `comment` text,
  `end_raw` int(14) NOT NULL,
  `session` char(50) NOT NULL,
  `rec_url` varchar(100) NOT NULL
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

