# $FreeBSD$

PORTNAME=	opam
PORTVERSION=	1.0.0
CATEGORIES=	devel

MASTER_SITES=	http://www.ocamlpro.com/pub/
DISTNAME=	${PORTNAME}-full-${PORTVERSION}

MAINTAINER=	ports@freebsd.org
COMMENT=	Package manager for OCaml

OPTIONS_DEFINE=		GIT RSYNC
OPTIONS_DEFAULT=	GIT RSYNC

GIT_DESC=		Git backend
RSYNC_DESC=		rsync backend

BUILD_DEPENDS=		ocaml>=3.12.1:${PORTSDIR}/lang/ocaml \
			curl:${PORTSDIR}/ftp/curl \
			gmake:${PORTSDIR}/devel/gmake

RUN_DEPENDS=		ocaml>=3.12.1:${PORTSDIR}/lang/ocaml \
			curl:${PORTSDIR}/ftp/curl

HAS_CONFIGURE=		yes
CONFIGURE_ARGS=		--prefix=${PREFIX} --mandir ${PREFIX}/man
USE_GMAKE=		yes

BINARIES=		opam opam-mk-repo

.for bin in ${BINARIES}
PLIST_FILES+=		bin/${bin}
.endfor

MAN1=			opam-check.1 opam-config.1 opam-help.1 opam-info.1 \
			opam-init.1 opam-install.1 opam-list.1 opam-mk-repo.1 \
			opam-pin.1 opam-reinstall.1 opam-remote.1 \
			opam-remove.1 opam-repository.1 opam-search.1 \
			opam-switch.1 opam-update.1 opam-upgrade.1 \
			opam-upload.1 opam.1

post-install:
.for bin in ${BINARIES}
	${STRIP_CMD} ${PREFIX}/bin/${bin}
.endfor

.include <bsd.port.options.mk>

.if ${PORT_OPTIONS:MGIT}
RUN_DEPENDS+=		git:${PORTSDIR}/devel/git
.endif

.if ${PORT_OPTIONS:MRSYNC}
RUN_DEPENDS+=		rsync:${PORTSDIR}/net/rsync
.endif

.include <bsd.port.mk>
