# stripchat-plugin
need help writing/rewriting stripchat.com streamlink plugin
 
 Hi
i have a program that record CB that is based on streamlink.
i tried finding a plugin that will enable the program/streamlink to work on stripchat.
i found one and the program finally started to look like its recording but the end file was not playable for some reason.
i tried troubleshooting it with ChatGPT but every suggestion to a better script or finding out where is the problem didn't go too well.

can someone help me with the script?? or write me a new one that works?
This is the script that i have:

import re

from streamlink.plugin import Plugin
from streamlink.plugin.api import validate
from streamlink.stream import HLSStream

_url_re = re.compile(r"https?://(\w+\.)?stripchat\.com/(?P<username>[a-zA-Z0-9_-]+)")

_post_schema = validate.Schema(
    {
        "cam": validate.Schema({
                    'streamName' : validate.text,
                    'viewServers': validate.Schema({'flashphoner-hls': validate.text})
        }),
        "user": validate.Schema({
                    'user' : validate.Schema({
                                'status' : validate.text,
                                'isLive' : bool
                    })
        })   
    }
)


class Stripchat(Plugin):
    @classmethod
    def can_handle_url(cls, url):
        return _url_re.match(url)

    def _get_streams(self):
        match = _url_re.match(self.url)
        username = match.group("username")
        api_call = "https://stripchat.com/api/front/v2/models/username/{0}/cam".format(username)
        headers = {
            "Content-Type": "application/x-www-form-urlencoded",
            "X-Requested-With": "XMLHttpRequest",
            "Referer": self.url,
        }

        res = self.session.http.get(api_call, headers=headers)
        data = self.session.http.json(res, schema=_post_schema)

        server = "https://b-{0}.doppiocdn.com/hls/{1}/master_{1}.m3u8".format(data["cam"]["viewServers"]["flashphoner-hls"],data["cam"]["streamName"])

        server0 = "https://b-{0}.doppiocdn.com/hls/{1}/{1}.m3u8".format(data["cam"]["viewServers"]["flashphoner-hls"],data["cam"]["streamName"])

        self.logger.info("Stream status: {0}".format(data["user"]["user"]["status"]))

        if (data["user"]["user"]["isLive"] is True and data["user"]["user"]["status"] == "public" and server):
            try:
                for s in HLSStream.parse_variant_playlist(self.session,server,headers={'Referer': self.url}).items():
                    yield s
            except IOError as err:
                stream = HLSStream(self.session, server0)
                yield "Auto", stream

__plugin__ = Stripchat


Thanks...
