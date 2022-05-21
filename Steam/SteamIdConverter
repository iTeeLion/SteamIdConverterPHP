<?php


namespace App\Steam;


class SteamIdConverter
{

    const STEAM_ID_64_BASE = '76561197960265728';
    const REGEX_STEAMID_ANYBASE = '/^STEAM_[0|1]:([0|1]:(.*))$/';
    const REGEX_STEAMID3 = '/\[?U:1:(\d+)\]?/';
    const REGEX_STEAMID64 = '/^(7656119)([0-9]{10})$/';
    const REGEX_STEAM_COMMUNITY_PROFILE = '/^https?:\/\/steamcommunity\.com\/profiles\/([0-9]+)\/?$/';
    const REGEX_STEAM_COMMUNITY_ID = '/^https?:\/\/steamcommunity\.com\/id\/(.*)\/?$/';

    public function convert($id)
    {
        $arResult = [];
        if (preg_match(self::REGEX_STEAMID_ANYBASE, $id)) {
            $arResult['type'] = 'steamid_anybase';
            $arResult['steamid_base'] = $this->steamIdAnyBaseToSteamIdBase($id);
        } elseif (preg_match(self::REGEX_STEAMID3, $id)) {
            $arResult['type'] = 'steamid3';
            $arResult['steamid32'] = $this->steamId3ToSteamId32($id);
            $arResult['steamid_base'] = $this->steamId32ToSteamIdBase($arResult['steamid32']);
        } elseif (preg_match(self::REGEX_STEAMID64, $id)) {
            $arResult['type'] = 'steamid64';
            $arResult['steamid_base'] = $this->steamId64ToSteamIdBase($id);
        } elseif (preg_match(self::REGEX_STEAM_COMMUNITY_PROFILE, $id)) {
            $arResult['type'] = 'steamcommunity_profile';
            $arResult['steamid64'] = $this->steamCommunityProfileToSteamId64($id);
            $arResult['steamid_base'] = $this->steamId64ToSteamIdBase($arResult['steamid64']);
        } elseif (preg_match(self::REGEX_STEAM_COMMUNITY_ID, $id)) {
            $arResult['type'] = 'steamcommunity_id';
            $arSteamCommunityInfo = $this->getSteamCommunityInfo($id . '?xml=1');
            if($arSteamCommunityInfo['success']){
                $arResult['steamcommunity_data'] = $arSteamCommunityInfo['data'];
                $arResult['steamid64'] = (string)$arResult['steamcommunity_data']->steamID64;
                $arResult['steamid_base'] = $this->steamId64ToSteamIdBase($arResult['steamid64']);
            }
        } elseif(preg_match('/^[0-9]+$/', $id)) {
            $arResult['type'] = 'steamid32';
            $arResult['steamid_base'] = $this->steamId32ToSteamIdBase($id);
        } else {
            $arResult['error'] = 'WTF you type in search field? o_O';
        }

        // fill converter data
        if(!$arResult['error']){
            if($arResult['steamid_base']){
                $arResult['steamid'] = $this->steamIdBaseToSteamId($arResult['steamid_base']);
                $arResult['steamid_modern'] = $this->steamIdBaseToSteamIdModern($arResult['steamid_base']);
                if (!$arResult['steamid32']) {
                    $arResult['steamid32'] = $this->steamIdBaseToSteamId32($arResult['steamid_base']);
                }
                if (!$arResult['steamid64']) {
                    $arResult['steamid64'] = $this->steamIdBaseToSteamId64($arResult['steamid_base']);
                }
                $arResult['steamid3'] = $this->steamId32ToSteamId3($arResult['steamid32']);
            }else{
                $arResult['error'] = 'Could not convert steamid!';
            }

            // get date from steamcommunity
            if (!$arSteamCommunityInfo) {
                $url = $this->getSteamCommunityInfoBySteamId64Url($arResult['steamid64']) . '?xml=1';
                $arSteamCommunityInfo = $this->getSteamCommunityInfo($url);
                if($arSteamCommunityInfo['success']){
                    $arResult['steamcommunity_data'] = $arSteamCommunityInfo['data'];
                }
            }

            if(!$arSteamCommunityInfo['success']){
                $arResult['warn'] = 'Could not load steamcommunity data! ' . $arSteamCommunityInfo['error'] ;
            }

            // fill steamcommunity data
            $arResult['steamcommunity_profile'] = $this->steamId64ToSteamCommunityProfile($arResult['steamid64']);
            if ($arResult['steamcommunity_data']->customURL) {
                $arResult['steamcommunity_id'] = $this->getSteamCommunityInfoProfileIdUrl((string)$arResult['steamcommunity_data']->customURL);
            }
        }

        return $arResult;
    }

    ###
    ### SteamIdBase
    ###

    protected function steamIdAnyBaseToSteamIdBase($id)
    {
        preg_match(self::REGEX_STEAMID_ANYBASE, $id, $res);
        return $res[1];
    }

    protected function steamIdBaseToSteamId($id)
    {
        return "STEAM_0:" . $id;
    }

    protected function steamIdBaseToSteamIdModern($id)
    {
        return "STEAM_1:" . $id;
    }

    protected function steamIdBaseToSteamId32($id)
    {
        $arId = explode(":", $id);
        if ($id % 2) {
            return $arId[1] * 2 + 1;
        }
        return $arId[1] * 2;
    }

    protected function steamIdBaseToSteamId64($id)
    {
        $arId = explode(':', $id);
        return bcadd(bcadd(bcmul($arId[1], '2'), self::STEAM_ID_64_BASE), $arId[0]);
    }

    ###
    ### SteamId3 to ...
    ###

    protected function steamId3ToSteamId32($id)
    {
        return preg_replace(self::REGEX_STEAMID3, "$1", $id);
    }

    ###
    ### SteamId32 to ...
    ###

    protected function steamId32ToSteamIdBase($id)
    {
        return $id % 2 . ":" . intval($id / 2);
    }

    protected function steamId32ToSteamId3($id)
    {
        return 'U:1:' . $id;
    }

    ###
    ### SteamId64 to ...
    ###

    protected function steamId64ToSteamIdBase($id)
    {
        if (bcmod($id, '2') == 0) {
            $idnum = '0';
            $temp = bcsub($id, self::STEAM_ID_64_BASE);
        } else {
            $idnum = '1';
            $temp = bcsub($id, bcadd(self::STEAM_ID_64_BASE, '1'));
        }
        $accnum = bcdiv($temp, '2');
        return $idnum . ":" . number_format($accnum, 0, '', '');
    }

    protected function steamId64ToSteamCommunityProfile($id)
    {
        return 'https://steamcommunity.com/profiles/' . $id . '/';
    }

    ###
    ### Steam community to ...
    ###

    protected function steamCommunityProfileToSteamId64($id)
    {
        preg_match(self::REGEX_STEAM_COMMUNITY_PROFILE, $id, $res);
        return $res[1];
    }

    ###
    ### Steam community
    ###

    protected function getSteamCommunityInfoBySteamId64Url($id)
    {
        return 'https://steamcommunity.com/profiles/' . $id . '/';
    }

    protected function getSteamCommunityInfoProfileIdUrl($id)
    {
        return 'https://steamcommunity.com/id/' . $id . '/';
    }

    protected function getSteamCommunityInfo($url)
    {
        $arResult = ['success' => true];

        try {
            $ch = curl_init();
            curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
            curl_setopt($ch, CURLOPT_URL, $url);
            $xmlData = curl_exec($ch);
            curl_close($ch);

            $arResult['data'] = new \SimpleXMLElement($xmlData);

            if(!empty((string)$arResult['data']->error)){
                $arResult['success'] = false;
                $arResult['error'] = (string)$arResult['data']->error;
            }
        } catch (\Exception $e) {
            $arResult['success'] = false;
            $arResult['error'] = $e->getMessage();
        }

        return $arResult;
    }

}
