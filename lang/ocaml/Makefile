# Created by: Kiriyama Kazuhiko <kiri@kiri.toba-cmt.ac.jp>
# $FreeBSD: http://svn.freebsd.org/ports/head/lang/ocaml/Makefile 319870 2013-06-04 15:13:42Z bapt $

PORTNAME=	ocaml
PORTVERSION=	4.00.1
CATEGORIES=	lang
MASTER_SITES=	http://caml.inria.fr/pub/distrib/${DISTNAME:R}/ \
		${MASTER_SITE_GENTOO}
MASTER_SITE_SUBDIR=	distfiles
DISTFILES=	${DISTNAME}${EXTRACT_SUFX}
EXTRACT_ONLY=	${DISTNAME}${EXTRACT_SUFX}

MAINTAINER?=	michipili@gmail.com
COMMENT?=	The Objective Caml compiler and programming environment

.if empty(PORT_OPTIONS:MDOCS)
DOCSDISTNAME=  ${DISTNAME:C/([[:digit:]]+)\.([[:digit:]]+)\.([[:digit:]]+)/\1.\2/}
DISTFILES+=    ${DOCSDISTNAME}-refman-html.tar.gz \
               ${DOCSDISTNAME}-refman.ps.gz \
               ${DOCSDISTNAME}-refman.pdf
.endif

USE_BZIP2=	yes
REINPLACE_ARGS=	-i ""
HAS_CONFIGURE=	yes
ALL_TARGET=	world.opt
STRIP=
MAKE_JOBS_UNSAFE=	yes

CONFIGURE_ARGS=		-verbose -prefix "${PREFIX}" \
			-cc "${CC} ${CFLAGS}" \
			-as "${AS} ${ASFLAGS}" \
			-aspp "${CC} -c" \
			-partialld "${LD} -r"

OPTIONS_DEFINE=		X11 TK THREADS
OPTIONS_DEFAULT=	X11 TK THREADS

TK_DESC=	LablTk library (requires X11 support)

MODOPT=		camlp4o camlp4r ocamlc ocamldep ocamldoc ocamllex ocamlopt
PATTERN=	[[:space:]]*(do|then)?[[:space:]]*)(\$$\(CP\)|cp)([[:space:]]

.include <bsd.port.pre.mk>

.if ${ARCH} == powerpc
ARCH=	power
.endif

.if ${ARCH} == power
PLIST_SUB+=	PROF="@comment "
.else
PLIST_SUB+=	PROF=""
.endif

.include "${FILESDIR}/manfiles"

.if !${PORT_OPTIONS:MDOCS}
PLIST_SUB+=    DOC=""
PORTDOCS =     htmlman ${DOCSDISTNAME}-refman.ps.gz ${DOCSDISTNAME}-refman.pdf
.else
PLIST_SUB+=    DOC="@comment "
.endif

.if ${PORT_OPTIONS:MTHREADS}
PLIST_SUB+=	THREADS=""
THR_CPP=	${PTHREAD_CFLAGS}
THR_LD=		${PTHREAD_LIBS}
.else
PLIST_SUB+=	THREADS="@comment "
THR_CPP=	#empty
THR_LD=		#empty
CONFIGURE_ARGS+=-no-pthread
.endif

.if ${PORT_OPTIONS:MX11}
PLIST_SUB+=	X11=""
USE_XORG=	x11
CONFIGURE_ARGS+=-x11include ${LOCALBASE}/include -x11lib ${LOCALBASE}/lib
.else
PLIST_SUB+=	X11="@comment "
WITHOUT_TK=	yes
CONFLICTS+=	ocaml-[0-9]* ocaml-notk-[0-9]*
CONFIGURE_ARGS+=-no-x11
.endif

.if ! ${PORT_OPTIONS:MTK}
PLIST_SUB+=	TK="@comment "
.if ${PORT_OPTIONS:MX11}
.endif
CONFIGURE_ARGS+=-no-tk
.else
USE_TK=		84+
INVALID_TK_VER =	86
.if ${PORT_OPTIONS:MTHREADS}
USE_TK_THREADS=		yes
.endif
.include "${PORTSDIR}/Mk/bsd.tcl.mk"
PLIST_SUB+=	TK=""
CONFLICTS+=	ocaml-nox11-[0-9]* ocaml-notk-[0-9]*
CONFIGURE_ARGS+=	-tkdefs \
			"-I${TCL_INCLUDEDIR} -I${TK_INCLUDEDIR} ${THR_CPP}" \
			-tklibs "-L${LOCALBASE}/lib -ltk${TK_VER:S|.||}${_TCL_THREADS_SUFFIX} \
			-ltcl${TCL_VER:S|.||}${_TCL_THREADS_SUFFIX} ${THR_LD}"
.endif

.if ${ARCH} == sparc64
BROKEN=	ocamlc.opt seg-faults on ${ARCH}, please try to fix
.endif

post-extract:
.if ${ARCH} == power
	${MV} ${WRKSRC}/asmrun/power-elf.S ${WRKSRC}/asmrun/power-bsd.S
.endif
.if !${PORT_OPTIONS:MDOCS}
	@if ! (cd ${WRKDIR} && \
	  ${TAR} xf ${_DISTDIR}${DOCSDISTNAME}-refman-html.tar.gz \
	  ) \
	  then \
	  exit 1; \
	  fi;
.endif

.if !defined(NOPORTDOCS)
	@if ! (cd ${WRKDIR} && \
	  ${TAR} xf ${_DISTDIR}${DOCSDISTNAME}-refman-html.tar.gz \
	) \
	then \
	  exit 1; \
	fi;
.endif

post-patch:
	@${REINPLACE_CMD} \
		-e '\|MKLIB=|s|ar rc|${AR} rc|' \
		-e '\|RANLIB|s|ranlib|${RANLIB}|' \
		-e '\|bytecclinkopts=|s|""|"${LDFLAGS}"|' \
		-e "\|nativecclinkopts=|s|''|\"${LDFLAGS:C/-Wl,//}\"|" \
		-e '\|mksharedlib=|s|$$bytecc|& $$bytecclinkopts|' \
		-e 's|-lbfd -ldl|-lbfd $$dllib|' \
		${WRKSRC}/configure
# don't hardcode gcc for depend target
	@${REINPLACE_CMD} -e 's|gcc|$$(CC)|' \
		${WRKSRC}/*/Makefile \
		${WRKSRC}/*/*/Makefile
.if ${PORT_OPTIONS:MTK}
	@${REINPLACE_CMD} -e 's|/usr/bin/wish|${WISH}|' \
		${WRKSRC}/otherlibs/labltk/examples_labltk/hello.tcl
.endif

post-configure:
# CFLAGS safeness
	@${REINPLACE_CMD} -E \
		-e 's|(BYTECCCOMPOPTS[ \t]*=.*)|\1 ${CFLAGS} ${THR_CPP}|' \
		-e 's|(NATIVECCCOMPOPTS[ \t]*=.*)|\1 ${CFLAGS} ${THR_CPP}|' \
		-e '/^PTHREAD_LINK/s,-pthread,${THR_LD},g' \
		${WRKSRC}/config/Makefile

# INSTALL* safeness (which preserves the correct permissions despite
# user's umask(1)
	@${FIND} ${WRKDIR} -type f -name Makefile -print0 | \
		${XARGS} -0 -n 5 -x ${REINPLACE_CMD} -E \
		-e 's,\$$\(ARCH\),${ARCH:sparc64=sparc},g'	\
		-e 's,^(${PATTERN}+.*INSTALLDIR),\1\$${BSD_INSTALL_DATA} \4,' \
		-e 's,^(${PATTERN}+.*BINDIR),\1\$${BSD_INSTALL_PROGRAM} \4,' \
		-e 's,^(${PATTERN}+.*LIBDIR),\1\$${BSD_INSTALL_DATA} \4,' \
		-e 's,(${PATTERN}+.*(MANDIR|MANODIR|MANEXT)),\1\$${BSD_INSTALL_MAN} \4,'

check test: install
	@cd ${WRKSRC}/testsuite; ${RM} -f _log; for d in tests/* ; do \
	${SETENV} ${MAKE_ENV} ${MAKE} ${_MAKE_JOBS} ${MAKE_ARGS} one DIR=$${d} \
	2>&1 | tee -a _log || ${TRUE} ; done ; \
	${SETENV} ${MAKE_ENV} ${MAKE} ${_MAKE_JOBS} ${MAKE_ARGS} report

post-install:
	@${TOUCH} ${PREFIX}/lib/ocaml/ocamldoc/custom/.keep
.if ${PORT_OPTIONS:MTK}
	@${MKDIR} ${EXAMPLESDIR}
	@(cd ${WRKSRC}/otherlibs/labltk && \
		${COPYTREE_SHARE} examples_labltk ${EXAMPLESDIR} \
		"! -name .cvsignore" && \
		${COPYTREE_SHARE} examples_camltk ${EXAMPLESDIR} \
		"! -name .cvsignore")
.endif

.if !${PORT_OPTIONS:MDOCS}
	${MKDIR} ${DOCSDIR}
	(cd ${WRKDIR} && ${COPYTREE_SHARE} htmlman ${DOCSDIR})
	${INSTALL_DATA} ${_DISTDIR}${DOCSDISTNAME}-refman.ps.gz ${DOCSDIR}
	${INSTALL_DATA} ${_DISTDIR}${DOCSDISTNAME}-refman.pdf ${DOCSDIR}
.endif

.for PROG in camlp4o.opt camlp4of.opt camlp4oof.opt camlp4orf.opt \
	camlp4r.opt camlp4rf.opt ocamlc.opt ocamldep.opt ocamldoc.opt \
	ocamllex.opt ocamlopt.opt ocamlrun ocamlyacc
	@${STRIP_CMD} ${PREFIX}/bin/${PROG}
.endfor

# Fix permissions for ld.conf
	@${CHMOD} 644 ${PREFIX}/lib/ocaml/ld.conf

.include <bsd.port.post.mk>
