<script lang="ts">
    import { onMount } from 'svelte';
    import { ShopMstsDoc, }  from '../../../graphql/generated';
    import { browser } from '$app/environment';
    import client from '../../../../lib/graphql/apollo';
    import { checkLoginStatus, currentUser } from '$lib/auth';
    import InputField from '$lib/components/InputField.svelte';
    import InputLabelHalf from '$lib/components/InputLabelHalf.svelte';
    import SelectField from '$lib/components/SelectField.svelte';
    import TextareaField from '$lib/components/TextareaField.svelte';
    import Button from '$lib/components/Button.svelte';
    import Heading from '$lib/components/Heading.svelte';
    import Hr from '$lib/components/Hr.svelte';
    import { CreateUserProfileDoc, CreateUserSnsDoc, UpdateUserSnsDoc, UpdateUserProfileDoc, UpdateUserDoc } from '../../../graphql/generated';
    import type { CreateUserProfileInput, UserSnsCreateInput, UpdateUserSnsInput, UpdateUserProfileInput, ShopMstUpdateOneWithoutUserProfilesInput, ShopMstConnectInput, UserUpdateInput } from '../../../graphql/generated';

    let name = '';
    let catchphrase = '';
    let introduction = '';
    let selectedShopCode = '';
    let x = '';
    let facebook = '';
    let instagram = '';
    let profileId: number | null = null;;
    let snsId: number | null = null;;

    let shopMsts: { code: string, name: string}[] = [];

    onMount(async () => {
        await checkLoginStatus();
        try {
          const { data } = await client.query({
            query: ShopMstsDoc,
          });
          $: shopMsts = data.shopMsts;
        } catch (error) {
          console.error('Error fetching shop msts:', error);
        }

        // ユーザ情報の取得
        const user = await currentUser();
        if (user) {
            name = user.name;
            if (user.userProfile) {
                profileId = user.userProfile.id || null;
                catchphrase = user.userProfile.catchphrase || '';
                introduction = user.userProfile.introduction || '';
                selectedShopCode = user.userProfile.shopMstCode || '';
            }
            if (user.userSns) {
                snsId = user.userSns.id || null;
                x = user.userSns.x || '';
                facebook = user.userSns.facebook || '';
                instagram = user.userSns.instagram || '';
            }
        }
    });

    // 更新
    async function update() {
      try {
        const user = await currentUser();
        const email = user.email;
        // ニックネームの更新
        const userUpdateData:UserUpdateInput = {
          name: name
        };
        const { data: userRes } = await client.mutate({
          mutation: UpdateUserDoc,
          variables: {
              id: user.id,
              data: userUpdateData
          }
        });
        console.log(userRes);

        if (!profileId) {
          // UserProfileの作成
          const userProfileData: CreateUserProfileInput = {
                  user: { connect: { email: email } },
                  shopMst: selectedShopCode ? { connect: { code: selectedShopCode } } : undefined,
                  catchphrase: catchphrase,
                  introduction: introduction
          };
          const { data: userProfileRes } = await client.mutate({
            mutation: CreateUserProfileDoc,
            variables: {
                data: userProfileData
            }
          });
          profileId = userProfileRes.createUserProfile.id;
        } else {
          // UserProfileの更新
          const shopData:ShopMstConnectInput = { code: selectedShopCode ? selectedShopCode : ''};
          const shopConnectData:ShopMstUpdateOneWithoutUserProfilesInput = { connect: shopData };
          const userProfileData :UpdateUserProfileInput = {
            catchphrase: catchphrase,
            introduction: introduction,
            shopMst: shopConnectData
          };
          const { data: userProfileRes } = await client.mutate({
            mutation: UpdateUserProfileDoc,
            variables: {
              id: profileId,
              data: userProfileData
            }
          });
          console.log(userProfileRes);
        }
        if (!snsId) {
          // UserSnsの作成
          const userSnsData: UserSnsCreateInput = {
                  user: { connect: { email: email } },
                  x: x,
                  facebook: facebook,
                  instagram: instagram
          };
          const { data: userSnsRes } = await client.mutate({
            mutation: CreateUserSnsDoc,
            variables: {
                data: userSnsData
            }
          });
          snsId = userSnsRes.createUserSns.id;
        } else {
          // UserSnsの更新
          const userSnsData :UpdateUserSnsInput = {
            x: x,
            facebook: facebook,
            instagram: instagram
          };
          const { data: userSnsRes } = await client.mutate({
            mutation: UpdateUserSnsDoc,
            variables: {
                id: profileId,
                data: userSnsData
            }
          });
          console.log(userSnsRes);
        }
        alert('更新が完了しました！');
      } catch (error) {
        // エラーハンドリング
        console.error('Update failed', error);
      }
    }

    // 前のページに戻る
    function goBack() {
        if (browser) {
            window.history.back();
        }
    }
</script>

<div class='relative flex-grow w-full max-w-xl mx-auto p-6 lg:p-8'>
    <Heading text="プロフィール編集" />
    <form on:submit|preventDefault={update}>
        <div style='display: flex; align-items: baseline;'>
            <InputLabelHalf forId="name" text="ニックネーム" />
            <InputField id="name" type="text" placeholder="はなこ" bind:value={name} required/>
        </div>
        <div style='display: flex; align-items: baseline;'>
            <InputLabelHalf forId="catchphrase" text="ひとこと" />
            <InputField id="catchphrase" type="text" placeholder="365日中本！" bind:value={catchphrase} required/>
        </div>
        <div style='display: flex; align-items: baseline;'>
            <InputLabelHalf forId="catchphrase" text="ホーム" />
            <SelectField id="shop" bind:value={selectedShopCode} options={shopMsts} required blank={true} />
        </div>
        <div style='display: flex; align-items: baseline;'>
            <InputLabelHalf forId="introduction" text="自己紹介" />
            <TextareaField id="introduction" placeholder="初めて中本に行ったのは二十歳の時。それから10年ずっと通い続けています。" bind:value={introduction} required />
        </div>
        <Hr/>
        <p class='text-gray-400 p-1.5 text-lg'>SNS</p>
        <div style='display: flex; align-items: baseline;'>
            <InputLabelHalf forId="introduction" text="X(旧Twitter)" />
            <InputField id="x" type="text" placeholder="@nakamoto" bind:value={x} />
        </div>
        <div style='display: flex; align-items: baseline;'>
            <InputLabelHalf forId="introduction" text="Facebook" />
            <InputField id="facebook" type="text" placeholder="10000000000000" bind:value={facebook} />
        </div>
        <div style='display: flex; align-items: baseline;'>
          <InputLabelHalf forId="introduction" text="インスタ" />
          <InputField id="instagram" type="text" placeholder="10000000000000" bind:value={instagram} />
      </div>
        <Button text="更新する" type='submit' disabled={!name} />
        <p class='text-center text-gray-400 cursor-pointer' on:click={goBack}>もどる</p>
    </form>
</div>
