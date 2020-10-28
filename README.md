Передавать конкретные параметры или объекты
===========================================

Совершенный код 7.5 "Советы по использованию параметров методов" стр.175
https://drive.google.com/file/d/1Vvve5NttbNakOkxzDSh1qp_Jr1t1PaW6/view?usp=sharing

39% всех ошибок это ошибки внутренних интерфейсов
http://www.lsmod.de/~bernhard/cvs/text/dipl/papers/p42-basili.pdf (Basili and Perricone/Computing practices 1984)

Передавать объекты это признак изменения объекта внутри функции
С++ Передача объекта в функцию по константной ссылке является нормой, если только не предполагается изменять объект внутри функции
https://foxford.ru/wiki/informatika/peredacha-ob-ektov-funktsiyam-v-s
Но в php const параметров нет, то есть любой параметр может быть изменён

Ограничивайте число параметров метода примерно семью 7 — магическое число. Психологические исследования показали, что люди, как правило, не могут следить более чем за семью элементами информации сразу
(Miller, 1956) https://ru.wikipedia.org/wiki/%D0%9C%D0%B0%D0%B3%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5_%D1%87%D0%B8%D1%81%D0%BB%D0%BE_%D1%81%D0%B5%D0%BC%D1%8C_%D0%BF%D0%BB%D1%8E%D1%81-%D0%BC%D0%B8%D0%BD%D1%83%D1%81_%D0%B4%D0%B2%D0%B0
Передавая объект, мы увеличиваем число параметров

Используйте все параметры
Исследования показали, что ошибки отсутствовали в 46% методов, не включавших неиспользуемых переменных
То есть 54% методов с неиспользуемыми переменными содержали ошибки
https://ntrs.nasa.gov/citations/19870015475 (Computer Science 1986)

Не используйте параметры метода в качестве рабочих переменных

```java
int Sample( int inputVal ) {
    inputVal = inputVal * CurrentMultiplier( inputVal );
    inputVal = inputVal + CurrentAdder( inputVal );
```

Правильно будет

```java
int Sample( int inputVal ) {
    int workingVal = inputVal;
    workingVal = workingVal * CurrentMultiplier( workingVal );
    workingVal = workingVal + CurrentAdder( workingVal );
```

пример плохого аргумента $invitation
из $invitation достается только id_invited_by
Передавая объект мы намекаем, что:
- $invitation может быть изменен и это повлияет на вызывающую функцию
- будем использовать весь объект

$codeOptions модифицируется, используется как рабочая переменная - плохо

```php
function enterReferralCode(Client $client, string $code, \stdClass $invitation, CouponSaveCodeOptions $codeOptions, bool $checkCard = false) {
        $this->checkActivateCodeByUser($client, $code, $invitation, $checkCard); // <- тут может показаться, что нужен объект $invitation, но это не так :)
        $codeOptions->id_referral_client = $invitation->id_invited_by;
        if (rand(1, 100) <= registry('coupon_ref_antifraud_percent')) {
            $this->checkAntiFraud($client, $codeOptions);
        }

        try {
            $couponCode = $this->getOldOrCreateNewCouponCode($invitation->id_invited_by, $code, $codeOptions->app_id);
            // В предыдущей операции возможна запись купона в мастер-базу, поэтому далее читаем тоже с мастера
            $ccOld = \Coupon::getCouponCode($couponCode->getCode(), false);
            $error = \Coupon::checkEnteredCode($ccOld, $client, $codeOptions);
            if ($error) {
                $exception = new CustomUserException($error['msg'] ?? null);
                $exception->setStatus($error['status'] ?? null);
                $exception->setTitle($error['title'] ?? null);
                throw $exception;
            }
            $result = \Coupon::saveCouponNew($ccOld, $client, $codeOptions);

            if (empty($result) || !$result || (isset($result['status']) && $result['status'] == 0)) {
                throw new Exception('Не удалось добавить купон');
            }
            $res = $this->insertClientInvitation($code, $client, $invitation->id_invited_by);
            if ($res === false) {
                throw new Exception('Не удалось добавить купон');
            }

            return $result;
        }
        catch (Throwable $exception) {
            throw $exception;
        }
    }
```

Первыми аргументами передавать самые важные параметры, тогда проще будет удалять лишние или давать им дефолтные значения.

```php
    public function getOldOrCreateNewCouponCode(int $idInvitedBy, string $code, ?int $appId = 0): Code
    {
        $coupon = $this->couponRepository->getCouponCodeByCode($code);
        if ($coupon) {
            return $coupon;
        }

        $appId = is_null($appId) ? 0 : intval($appId);
        $idReferralCoupon = self::getReferralCouponIdByApp($appId, $code);
```
