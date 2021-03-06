# libguestfs OCaml bindings
# Copyright (C) 2009 Red Hat Inc.
#
# This program is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

include $(top_srcdir)/subdir-rules.mk

generator_built = \
	guestfs.mli \
	guestfs.ml \
	guestfs-c-actions.c \
	$(srcdir)/bindtests.ml

EXTRA_DIST = \
	$(generator_built) \
	.depend \
	guestfs-c.c guestfs-c.h \
	html/.gitignore \
	META.in \
	run-bindtests \
	t/*.ml

CLEANFILES = *.cmi *.cmo *.cmx *.cma *.cmxa *.o *.a *.so
CLEANFILES += t/*.cmi t/*.cmo t/*.cmx t/*.o t/*.a t/*.so

if HAVE_OCAML

OCAMLCFLAGS = -g -warn-error CDEFLMPSUVYZX
OCAMLOPTFLAGS = $(OCAMLCFLAGS)

noinst_DATA = mlguestfs.cma mlguestfs.cmxa META

# Build the C part into a library, so that automake handles the C
# compilation step for us.  Note that we don't directly use this
# library; we link with the object files that it generates.
noinst_LIBRARIES = libguestfsocaml.a

# Note that both calls to ocamlmklib below will create
# 'libmlguestfs.a' and if run at the same time, they will stomp on or
# corrupt each others copy.  Hence we have to serialize the calls.

CLEANFILES += stamp-mlguestfs

mlguestfs.cma mlguestfs.cmxa: stamp-mlguestfs

stamp-mlguestfs: libguestfsocaml.a guestfs.cmo guestfs.cmx
	$(OCAMLMKLIB) -o mlguestfs \
	  $(libguestfsocaml_a_OBJECTS) guestfs.cmo \
	  -L$(top_builddir)/src/.libs -lguestfs
	$(OCAMLMKLIB) -o mlguestfs \
	  $(libguestfsocaml_a_OBJECTS) guestfs.cmx \
	  -L$(top_builddir)/src/.libs -lguestfs
	touch $@

libguestfsocaml_a_CPPFLAGS = \
	-DGUESTFS_PRIVATE=1 \
	-I$(top_builddir) -I$(OCAMLLIB) -I$(top_srcdir)/ocaml \
	-I$(top_srcdir)/src -I$(top_builddir)/src \
	-I$(top_srcdir)/gnulib/lib -I../gnulib/lib

libguestfsocaml_a_CFLAGS = \
	$(WARN_CFLAGS) $(WERROR_CFLAGS) \
	-fPIC

libguestfsocaml_a_SOURCES = \
	guestfs-c.c \
	guestfs-c-actions.c \
	../src/utils.c

if HAVE_OCAMLDOC

noinst_DATA += html/index.html

html/index.html: $(srcdir)/guestfs.mli $(srcdir)/guestfs.ml
	-$(OCAMLDOC) -d html -html $^

endif

TESTS_ENVIRONMENT = $(top_builddir)/run --test $(VG)

test_progs_bc = \
	t/guestfs_010_load.bc \
	t/guestfs_020_create.bc \
	t/guestfs_030_create_flags.bc \
	t/guestfs_040_create_multiple.bc \
	t/guestfs_050_handle_properties.bc \
	t/guestfs_060_explicit_close.bc \
	t/guestfs_070_optargs.bc \
	t/guestfs_410_close_event.bc \
	t/guestfs_420_log_messages.bc

test_progs_opt = \
	t/guestfs_010_load.opt \
	t/guestfs_020_create.opt \
	t/guestfs_030_create_flags.opt \
	t/guestfs_040_create_multiple.opt \
	t/guestfs_050_handle_properties.opt \
	t/guestfs_060_explicit_close.opt \
	t/guestfs_070_optargs.opt \
	t/guestfs_410_close_event.opt \
	t/guestfs_420_log_messages.opt

if ENABLE_APPLIANCE
test_progs_bc += \
	t/guestfs_100_launch.bc \
	t/guestfs_430_progress_messages.bc
test_progs_opt += \
	t/guestfs_100_launch.opt \
	t/guestfs_430_progress_messages.opt
endif

test_progs_all = $(test_progs_bc) $(test_progs_opt)

TESTS = run-bindtests $(test_progs_all)

noinst_DATA += bindtests.bc bindtests.opt $(test_progs_all)

%.bc: %.cmo mlguestfs.cma
	$(top_builddir)/libtool -dlopen $(top_builddir)/src/.libs/libguestfs.la --mode=execute \
	$(OCAMLFIND) ocamlc $(OCAMLCFLAGS) -I . -package unix -linkpkg mlguestfs.cma $< -o $@

%.opt: %.cmx mlguestfs.cmxa
	$(OCAMLFIND) ocamlopt $(OCAMLOPTFLAGS) -cclib -L$(top_builddir)/src/.libs -I . -package unix -linkpkg mlguestfs.cmxa $< -o $@

check-valgrind:
	$(MAKE) VG="$(top_builddir)/run @VG@" check

# Dependencies.
%.cmi: %.mli
	$(OCAMLFIND) ocamlc $(OCAMLCFLAGS) -package unix -c $< -o $@
guestfs.cmo: guestfs.ml
	$(OCAMLFIND) ocamlc $(OCAMLCFLAGS) -package unix -c $< -o $@
%.cmo: %.ml mlguestfs.cma
	$(OCAMLFIND) ocamlc $(OCAMLCFLAGS) -package unix -c $< -o $@
guestfs.cmx: guestfs.ml
	$(OCAMLFIND) ocamlopt $(OCAMLOPTFLAGS) -package unix -c $< -o $@
%.cmx: %.ml mlguestfs.cmxa
	$(OCAMLFIND) ocamlopt $(OCAMLOPTFLAGS) -package unix -c $< -o $@

depend: .depend

.depend: $(wildcard $(abs_srcdir)/*.mli) $(wildcard $(abs_srcdir)/*.ml) $(wildcard $(abs_srcdir)/t/*.ml)
	rm -f $@ $@-t
	$(OCAMLFIND) ocamldep -I ../ocaml -I $(abs_srcdir) $^ | \
	  $(SED) 's/ *$$//' | \
	  $(SED) -e :a -e '/ *\\$$/N; s/ *\\\n */ /; ta' | \
	  $(SED) -e 's,$(abs_srcdir)/,$(builddir)/,g' | \
	  sort > $@-t
	mv $@-t $@

-include .depend

SUFFIXES = .cmo .cmi .cmx .ml .mli .mll .mly

# Do the installation by hand, because we want to run ocamlfind.
install-data-hook:
	mkdir -p $(DESTDIR)$(OCAMLLIB)
	mkdir -p $(DESTDIR)$(OCAMLLIB)/stublibs
	$(OCAMLFIND) install \
	  -ldconf ignore -destdir $(DESTDIR)$(OCAMLLIB) \
	  guestfs \
	  META *.so *.a *.cma *.cmx *.cmxa *.cmi $(srcdir)/*.mli
	rm $(DESTDIR)$(OCAMLLIB)/guestfs/bindtests.*
	rm $(DESTDIR)$(OCAMLLIB)/guestfs/libguestfsocaml.a

CLEANFILES += $(noinst_DATA)

endif
