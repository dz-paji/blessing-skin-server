<template>
  <section class="content">
    <div class="row">
      <div class="col-md-6">
        <div class="box box-primary">
          <div class="box-body table-responsive no-padding">
            <table class="table table-hover">
              <thead>
                <tr>
                  <th>PID</th>
                  <th v-t="'general.player.player-name'" />
                  <th v-t="'user.player.operation'" />
                </tr>
              </thead>

              <tbody>
                <tr
                  v-for="(player, index) in players"
                  :key="player.pid"
                  class="player"
                  :class="{ 'player-selected': player.pid === selected }"
                  @click="preview(player)"
                >
                  <td class="pid">{{ player.pid }}</td>
                  <td class="player-name">{{ player.name }}</td>
                  <td>
                    <el-button size="medium" @click="changeName(player)">
                      {{ $t('user.player.edit-pname') }}
                    </el-button>
                    <el-button
                      size="medium"
                      type="warning"
                      data-toggle="modal"
                      data-target="#modal-clear-texture"
                    >
                      {{ $t('user.player.delete-texture') }}
                    </el-button>
                    <el-button
                      size="medium"
                      type="danger"
                      @click="deletePlayer(player, index)"
                    >
                      {{ $t('user.player.delete-player') }}
                    </el-button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
          <div class="box-footer clearfix">
            <el-button type="primary" data-toggle="modal" data-target="#modal-add-player">
              <i class="fas fa-plus" aria-hidden="true" /> &nbsp;{{ $t('user.player.add-player') }}
            </el-button>
          </div>
        </div>

        <div v-once class="box box-default">
          <div class="box-header with-border">
            <h3 v-t="'general.tip'" class="box-title" />
          </div><!-- /.box-header -->
          <div class="box-body">
            <p v-t="'user.player.login-notice'" />
          </div><!-- /.box-body -->
        </div><!-- /.box -->
      </div>

      <div class="col-md-6">
        <previewer
          v-if="using3dPreviewer"
          :skin="skinUrl"
          :cape="capeUrl"
          title="user.player.player-info"
        >
          <template #footer>
            <el-button data-test="to2d" @click="togglePreviewer">
              {{ $t('user.switch2dPreview') }}
            </el-button>
          </template>
        </previewer>
        <div v-else class="box">
          <div class="box-header with-border">
            <!-- eslint-disable-next-line vue/no-v-html -->
            <h3 class="box-title" v-html="$t('user.player.player-info')" />
          </div>
          <div class="box-body">
            <div id="preview-2d">
              <p>
                {{ $t('general.skin') }}
                <a v-if="preview2d.skin" :href="`${baseUrl}/skinlib/show/${preview2d.skin}`">
                  <img
                    class="skin2d"
                    :src="`${baseUrl}/preview/64/${preview2d.skin}.png`"
                  >
                </a>
                <span v-else v-t="'user.player.texture-empty'" class="skin2d" />
              </p>

              <p>
                {{ $t('general.cape') }}
                <a v-if="preview2d.cape" :href="`${baseUrl}/skinlib/show/${preview2d.cape}`">
                  <img
                    class="skin2d"
                    :src="`${baseUrl}/preview/64/${preview2d.cape}.png`"
                  >
                </a>
                <span v-else v-t="'user.player.texture-empty'" class="skin2d" />
              </p>
            </div>
          </div><!-- /.box-body -->
          <div class="box-footer">
            <el-button @click="togglePreviewer">
              {{ $t('user.switch3dPreview') }}
            </el-button>
          </div>
        </div><!-- /.box -->
      </div>
    </div>

    <add-player-dialog @add="fetchPlayers" />

    <div
      id="modal-clear-texture"
      class="modal fade"
      tabindex="-1"
      role="dialog"
    >
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <button
              type="button"
              class="close"
              data-dismiss="modal"
              aria-label="Close"
            >
              <span aria-hidden="true">&times;</span>
            </button>
            <h4 v-t="'user.chooseClearTexture'" class="modal-title" />
          </div>
          <div class="modal-body">
            <label class="form-group">
              <input v-model="clear.skin" type="checkbox"> {{ $t('general.skin') }}
            </label>
            <br>
            <label class="form-group">
              <input v-model="clear.cape" type="checkbox"> {{ $t('general.cape') }}
            </label>
          </div>
          <div class="modal-footer">
            <el-button data-dismiss="modal">{{ $t('general.close') }}</el-button>
            <el-button type="primary" data-test="clearTexture" @click="clearTexture">
              {{ $t('general.submit') }}
            </el-button>
          </div>
        </div><!-- /.modal-content -->
      </div><!-- /.modal-dialog -->
    </div>
  </section>
</template>

<script>
import AddPlayerDialog from '../../components/AddPlayerDialog.vue'

export default {
  name: 'Players',
  components: {
    AddPlayerDialog,
    Previewer: () => import('../../components/Previewer.vue'),
  },
  props: {
    baseUrl: {
      type: String,
      default: blessing.base_url,
    },
  },
  data() {
    return {
      players: [],
      selected: 0,
      using3dPreviewer: true,
      skinUrl: '',
      capeUrl: '',
      preview2d: {
        skin: 0,
        cape: 0,
      },
      clear: {
        skin: false,
        cape: false,
      },
    }
  },
  beforeMount() {
    this.fetchPlayers()
  },
  methods: {
    async fetchPlayers() {
      this.players = (await this.$http.get('/user/player/list')).data
    },
    togglePreviewer() {
      this.using3dPreviewer = !this.using3dPreviewer
    },
    async preview(player) {
      this.selected = player.pid

      this.preview2d.skin = player.tid_skin
      this.preview2d.cape = player.tid_cape

      if (player.tid_skin) {
        const { data: skin } = await this.$http.get(`/skinlib/info/${player.tid_skin}`)
        this.skinUrl = `${this.baseUrl}/textures/${skin.hash}`
      } else {
        this.skinUrl = ''
      }
      if (player.tid_cape) {
        const { data: cape } = await this.$http.get(`/skinlib/info/${player.tid_cape}`)
        this.capeUrl = `${this.baseUrl}/textures/${cape.hash}`
      } else {
        this.capeUrl = ''
      }
    },
    async changeName(player) {
      let value
      try {
        ({ value } = await this.$prompt(this.$t('user.changePlayerName'), {
          inputValue: player.name,
          inputValidator: val => !!val || this.$t('user.emptyPlayerName'),
        }))
      } catch {
        return
      }

      const { code, message } = await this.$http.post(
        `/user/player/rename/${player.pid}`,
        { name: value }
      )
      if (code === 0) {
        this.$message.success(message)
        player.name = value
      } else {
        this.$message.warning(message)
      }
    },
    async clearTexture() {
      if (Object.values(this.clear).every(value => !value)) {
        return this.$message.warning(this.$t('user.noClearChoice'))
      }

      const { code, message } = await this.$http.post(
        `/user/player/texture/clear/${this.selected}`,
        this.clear
      )
      if (code === 0) {
        $('.modal').modal('hide')
        this.$message.success(message)
        const player = this.players.find(({ pid }) => pid === this.selected)
        Object.keys(this.clear)
          .filter(type => this.clear[type])
          .forEach(type => (player[`tid_${type}`] = 0))
      } else {
        this.$message.warning(message)
      }
    },
    async deletePlayer(player, index) {
      try {
        await this.$confirm(this.$t('user.deletePlayerNotice'), {
          title: this.$t('user.deletePlayer'),
          type: 'warning',
        })
      } catch {
        return
      }

      const { code, message } = await this.$http.post(`/user/player/delete/${player.pid}`)
      if (code === 0) {
        this.$delete(this.players, index)
        this.$message.success(message)
      } else {
        this.$message.warning(message)
      }
    },
  },
}
</script>

<style lang="stylus">
.player
  cursor pointer
  border-bottom 1px solid #f4f4f4

  .pid, .player-name
    padding-top 13px

.player:last-child
  border-bottom none

.player-selected
  background-color #f5f5f5

.skin2d
  float right
  max-height 64px
  width 64px
  font-size 16px

#preview-2d > p
  height 64px
  line-height 64px

.form-group
  cursor pointer
</style>
