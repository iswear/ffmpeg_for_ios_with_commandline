# ffmpeg_for_ios_with_commandline
The project provide the detail steps for you to compile ffmpeg to iOS with commendline supported. The complie script was from https://github.com/kewlbear/FFmpeg-iOS-build-script. Thank you!

##the modification of source code and some other file
1.copy these files and folders to libutil (ffmpeg.h ffmpeg.c ffmpeg_opt.c ffmpeg_filter.c cmdutils.h cmdutils_common_opts.h cmdutils_opencl.c cmdutils.c compat)in fact, I copy all *.h and *.c files in ffmpeg and compat to libutil but the really userful files should be these I said above;   
2.modify the source code in ffmpeg.h   
	add code as follows:   
		#include <setjmp.h>   
		extern jmp_buf exit_buf;   
		int ffmpeg_main(int argc, char **argv);   
		
3.modify the source code in ffmpeg.c   
	add code as follows:   
		jmp_buf exit_buf;   
		   
	in the end of function ffmpeg_cleanup   
	add code as follows:   
		nb_input_streams = 0;   
		nb_input_files   = 0;   
		nb_output_streams = 0;   
		nb_output_files   = 0;   
		nb_filtergraphs = 0;   
		   
	rename the function main to ffmpeg_main the modify the code like this:   
		if(setjmp(exit_buf)){   
			return 0;   
		}else{   
			old code balabala   
			....   
		} 

4.modify the source code in cmdutils.h   
	add code as follows:   
		#include <setjmp.h>   
		extern jmp_buf exit_buf;   
		
5.modify the source code in cmdutils.c   
	replace the code of function exit_program   
		if (program_exit)   
			program_exit(ret);   
		longjmp(exit_buf, 1);   
		
6.modify the source code in ffmpeg_opt.c   
	comment some code in function assert_file_overwrite like this   
	/*   
		if (stdin_interaction && !no_file_overwrite) {   
			fprintf(stderr,"File '%s' already exists. Overwrite ? [y/N] ", filename);   
			fflush(stderr);   
			term_exit();   
			signal(SIGINT, SIG_DFL);   
			if (!read_yesno()) {   
				av_log(NULL, AV_LOG_FATAL, "Not overwriting - exiting\n");   
				exit_program(1);   
			}   
			term_init();   
		} else {  
	*/   
          	av_log(NULL, AV_LOG_FATAL, "File '%s' already exists. Exiting.\n", filename);   
          	exit_program(1);   
        //}   
        
7.modify the Makefile in libavutil as follows   
	add objs:   
		cmdutils.o                                               \   
		ffmpeg.o                                                 \   
		ffmpeg_opt.o                                             \   
		ffmpeg_filter.o                                          \   
	add headers:   
		ffmpeg.h                                                 \   
		
##compile   
	run the script ./build-ffmpeg.sh (the arches you wanted)   
	add the lib and headers to you project and add extern "C" {} for the header files you want to use   
	   
##besides   
	I'm not good at Makefile, so I borrow the libavutil. You can write a single one , that would be much better, I think. There is a compiled lib in the projects and a test commandline class FFMpegCommand, All those was compiled in XCODE 7.2.I think there maybe some memory leak in the code. It exits after end running before we modify the source code,And now it doesn't. If you found it, I think you have to modify it by yourself. If time allowed, I will check it. I think it must be a hard job. so good luck!

##References   
https://github.com/kewlbear/FFmpeg-iOS-build-script   
http://blog.sina.com.cn/s/blog_47522f7f0102vbwp.html   
https://www.baidu.com/link?url=PNf6SHdZTK9ipxx56UsKmm07Y8QBhWhQHULiPO0Ra3dsqkoL9E0D3u7918M-iressF2IlyE2Tk-b8Quh5XDesK&wd=&eqid=cea1d6ae00020e910000000356a9c4b3   
http://stackoverflow.com/questions/25742626/ffmpeg-link-error   
http://www.cnblogs.com/yc_sunniwell/archive/2010/07/14/1777431.html


