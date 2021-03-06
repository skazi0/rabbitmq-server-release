SOURCE_DIST_FILE ?= $(wildcard ../../../rabbitmq-server-*.tar.xz)

ifneq ($(filter-out clean,$(MAKECMDGOALS)),)
ifeq ($(SOURCE_DIST_FILE),)
$(error Cannot find source archive; please specify SOURCE_DIST_FILE)
endif
ifneq ($(words $(SOURCE_DIST_FILE)),1)
$(error Multiple source archives found; please specify SOURCE_DIST_FILE)
endif

VERSION ?= $(patsubst rabbitmq-server-%.tar.xz,%,$(notdir $(SOURCE_DIST_FILE)))
ifeq ($(VERSION),)
$(error Cannot determine version; please specify VERSION)
endif
endif

SOURCE_DIR = rabbitmq-server-$(VERSION)
TARGET_DIR = rabbitmq_server-$(VERSION)
TARGET_TARBALL = rabbitmq-server-generic-unix-$(VERSION)

unexport DEPS_DIR
unexport ERL_LIBS
MAKEOVERRIDES =

all: dist
	@:

dist:
# Log Erlang version.
	@echo
	@echo '--------------------------------------------------'
	@echo "Erlang version used to compile:"
	@erl -A0 -noinput -eval 'io:format("~s", [erlang:system_info(system_version)]), halt().'
	@echo '--------------------------------------------------'
	@echo
	xzcat $(SOURCE_DIST_FILE) | tar -xf -

# web-manpages are not used by generic-unix but by `make release` in the.
# Umbrella. Those manpages are copied to www.rabbitmq.com
	$(MAKE) -C $(SOURCE_DIR) \
		PREFIX= RMQ_ROOTDIR= \
		RMQ_ERLAPP_DIR=`pwd`/$(TARGET_DIR) \
		MANDIR=`pwd`/$(TARGET_DIR)/share/man \
		manpages web-manpages install install-man

	sed -e 's:^SYS_PREFIX=$$:SYS_PREFIX=\$${RABBITMQ_HOME}:' \
        $(TARGET_DIR)/sbin/rabbitmq-defaults >$(TARGET_DIR)/sbin/rabbitmq-defaults.tmp \
        && mv $(TARGET_DIR)/sbin/rabbitmq-defaults.tmp $(TARGET_DIR)/sbin/rabbitmq-defaults
	chmod 0755 $(TARGET_DIR)/sbin/rabbitmq-defaults

	mkdir -p $(TARGET_DIR)/etc/rabbitmq

	find $(TARGET_DIR) -print0 | LC_COLLATE=C sort -z | \
	xargs -0 tar --no-recursion -cf - | \
	xz > $(CURDIR)/$(TARGET_TARBALL).tar.xz

	if test "$(PACKAGES_DIR)"; then \
		mkdir -p "$(PACKAGES_DIR)"; \
		mv $(TARGET_TARBALL).tar.xz "$(PACKAGES_DIR)"; \
	fi

clean: clean_partial
	rm -f rabbitmq-server-generic-unix-*.tar.xz

clean_partial:
	rm -rf rabbitmq-server-* rabbitmq_server-*
